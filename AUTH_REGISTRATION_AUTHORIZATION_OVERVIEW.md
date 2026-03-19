# Cognito Signup and Signin Flow with Lambda Triggers

## Overview
This document explains the complete signup and signin flow using Amazon Cognito with two Lambda triggers:

1. Pre Sign-up trigger: validates that only company email addresses can register.
2. Post Confirmation trigger: inserts the newly confirmed user into DynamoDB.

Both Lambda functions are assumed to be already attached to the Cognito User Pool.

## Components in the Flow
1. Client App
   Sends signup and signin requests to Cognito.

2. Cognito User Pool
   Handles user creation, email verification, and login.

3. Pre Sign-up Lambda
   Runs before Cognito creates a user.
   Validates email domain.

4. Post Confirmation Lambda
   Runs after user confirms signup.
   Inserts user profile data into DynamoDB.

5. DynamoDB
   Stores user profile/authorization metadata for application use.

## Lambda Trigger 1: Pre Sign-up Domain Check
Purpose: allow signup only from lamipak.biz email addresses.

const domain = "lamipak.biz";
exports.handler = async (event) => {
    const email = event.request.userAttributes.email;

    if (event.triggerSource === "PreSignUp_SignUp") {
        if (!email || !email.endsWith(`@${domain}`)) {
            throw new Error("Email must be from lamipak.biz domain");
        }
    }

    // If the email is valid, continue the signup process
    return event;
};

What it does:
1. Reads email from the Cognito event.
2. Rejects signup if email is missing or not in allowed domain.
3. Returns the event when validation passes.

## Lambda Trigger 2: Post Confirmation User Insert
Purpose: create a user record in DynamoDB after successful email confirmation.

import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { PutCommand, DynamoDBDocumentClient } from "@aws-sdk/lib-dynamodb";

// Initialize DynamoDB Document Client
const client = new DynamoDBClient({ region: "us-east-1" });
const dynamoDb = DynamoDBDocumentClient.from(client);

export const handler = async (event) => {
    const email = event.request.userAttributes.email;
    const userId = event.request.userAttributes.sub;
    const signupDate = new Date().toISOString();

    // Check if this is the post-confirmation trigger
    if (event.triggerSource === "PostConfirmation_ConfirmSignUp") {
        const params = {
            TableName: "lamiUser",
            Item: {
                userId: userId,
                email: email,
                signupDate: signupDate
            }
        };

        try {
            // Use PutCommand to add the user data to DynamoDB
            await dynamoDb.send(new PutCommand(params));
            console.log("User added to DynamoDB:", params.Item);
        } catch (error) {
            console.error("Error adding user to DynamoDB:", error);
            throw error;  // Re-throw the error to handle it as an execution failure
        }
    }

    // Return the event if all succeeds
    return event;
};

What it does:
1. Executes only after signup confirmation.
2. Reads user attributes from Cognito event.
3. Creates a default user profile document.
4. Writes to DynamoDB with idempotency guard (no duplicate by userID).
5. Returns event to finish trigger flow.

## End-to-End Signup Flow
1. User enters email and password in client app.
2. Client calls Cognito SignUp API.
3. Cognito invokes Pre Sign-up Lambda.
4. Pre Sign-up Lambda checks email domain.
5. If domain is invalid, signup fails with error.
6. If domain is valid, Cognito creates user in Unconfirmed state.
7. Cognito sends verification code to email.
8. User confirms signup using code.
9. Cognito marks user as Confirmed.
10. Cognito invokes Post Confirmation Lambda.
11. Post Confirmation Lambda inserts user profile in DynamoDB.
12. Signup flow is complete.

## End-to-End Signin Flow
1. User enters email and password in client app.
2. Client calls Cognito SignIn API.
3. Cognito validates credentials and account status.
4. If valid, Cognito returns tokens (ID token, Access token, Refresh token).
5. Client establishes authenticated session.
6. Client can now call protected application functionality.

## Why DynamoDB Was Used for This Auth + CRUD Design
For this application, DynamoDB is a practical fit for both the Cognito user lifecycle and user/profile CRUD operations.

1. No extra database driver burden in Lambda
  Lambda can directly use the AWS SDK modules for DynamoDB that are available in the Lambda environment, so trigger code stays lightweight and simple. This avoids managing separate database client stacks and connection libraries typically needed for RDS-based access.

2. Better cost profile for this workload
  DynamoDB supports pay-per-request/serverless usage patterns, which is usually more cost efficient for variable or moderate traffic. Unlike RDS, there is no always-on database instance cost just to keep the service running.

3. Current data model does not need complex relational reporting
  The flow mainly needs straightforward operations such as create user after confirmation, fetch by userID/loginID/email, update profile/role fields, and delete/deactivate records. This key-based CRUD pattern is a strong DynamoDB use case.

4. Scales automatically with auth spikes
  Signup/signin traffic can be bursty (for example, onboarding waves). DynamoDB handles sudden throughput changes without manual capacity planning that is more common in relational setups.

5. Low-latency lookups for identity paths
  Authentication-adjacent APIs usually need quick reads by known keys (userID, loginID, email). DynamoDB is optimized for these predictable access patterns and provides consistent, fast performance.

6. Operational simplicity for a serverless architecture
  With DynamoDB, there is no DB server patching, instance sizing, connection pool tuning, or failover management in application code. This aligns well with Cognito + Lambda event-driven architecture.

7. High availability and durability by default
  DynamoDB is managed and replicated across Availability Zones, which reduces operational risk and improves reliability for user identity records.

8. Fine-grained security integration
  IAM-based permissions can tightly control which Lambda can read/write which table actions, keeping auth-related data access explicit and auditable.

In short, DynamoDB is a strong fit here because the application primarily needs reliable, low-latency, key-based CRUD around user identity records, not complex relational joins or heavy analytical reporting.

## Failure Cases and Expected Behavior
1. Wrong email domain at signup:
   Pre Sign-up Lambda throws error and user is not created.

2. User does not confirm email:
   Account remains Unconfirmed, signin does not complete successfully.


## Sequence Summary
1. SignUp request enters Cognito.
2. Pre Sign-up Lambda validates domain.
3. User confirms email.
4. Post Confirmation Lambda creates app user record.
5. SignIn returns tokens and starts session.

This is the complete independent flow for Cognito-based signup and signin with the two Lambda trigger extensions.
