# Work Report Usage Statistics Dashboard - Technical Specifications

## System Requirements

### Backend Requirements
- **Framework**: FastAPI (already in use)
- **Database**: AWS DynamoDB (already in use)
- **Python Version**: 3.7+
- **Dependencies**: Uses existing imports (boto3, decimal, datetime, etc.)

### Frontend Requirements
- **Framework**: React (already in use)
- **Node.js**: v14+ (already in use)
- **Browser**: Modern (ES6 support)
- **Dependencies**: No new npm packages required

## API Specifications

### Endpoint 1: GET /management/stats/usage/inactive-users

**Authorization**: 
- Requires `requesterEmail` parameter
- User must have `can_manage_reports` OR `can_manage_users` permission

**URL Format**:
```
GET /management/stats/usage/inactive-users?requesterEmail=user@domain.com&days=30
```

**Query Parameters**:
```
requesterEmail: string (EmailStr) - REQUIRED - User's email address
days: integer - OPTIONAL - Days to consider as "inactive threshold" (default: 30, min: 1, max: 365)
```

**Response Status Codes**:
- `200 OK` - Successfully retrieved inactive users
- `403 Forbidden` - User lacks permission (can_manage_reports or can_manage_users)
- `500 Internal Server Error` - DynamoDB error or processing error

**Response Body (200 OK)**:
```json
{
  "totalInactiveUsers": 42,
  "daysThreshold": 30,
  "users": [
    {
      "userId": "user-uuid-1",
      "email": "john.doe@company.com",
      "displayName": "John Doe",
      "clusterId": "cluster-id-123",
      "clusterName": "Engineering",
      "lastReportDate": "2024-01-15T10:30:00Z",
      "daysInactive": 34
    },
    {
      "userId": "user-uuid-2",
      "email": "jane.smith@company.com",
      "displayName": "Jane Smith",
      "clusterId": "cluster-id-456",
      "clusterName": "Operations",
      "lastReportDate": null,
      "daysInactive": 30
    }
  ]
}
```

**Error Response (403 Forbidden)**:
```json
{
  "detail": "You do not have permission to view usage statistics."
}
```

**Data Source**:
- Primary: `lamiUserWorkReport` table (scan all items)
- Secondary: `allLamiUsers` table (get user details)
- Filter: Cluster access control applied

**Response Time**: <3 seconds (typical for <1000 users)

---

### Endpoint 2: GET /management/stats/usage/top-bottom-users

**Authorization**: 
- Requires `requesterEmail` parameter
- User must have `can_manage_reports` OR `can_manage_users` permission

**URL Format**:
```
GET /management/stats/usage/top-bottom-users?requesterEmail=user@domain.com&limit=10
```

**Query Parameters**:
```
requesterEmail: string (EmailStr) - REQUIRED - User's email address
limit: integer - OPTIONAL - Number of top/bottom users (default: 10, min: 1, max: 50)
```

**Response Status Codes**:
- `200 OK` - Successfully retrieved users
- `403 Forbidden` - User lacks permission
- `500 Internal Server Error` - DynamoDB error

**Response Body (200 OK)**:
```json
{
  "topUsers": [
    {
      "userId": "user-uuid-1",
      "email": "alice@company.com",
      "displayName": "Alice Johnson",
      "clusterId": "cluster-id-123",
      "clusterName": "Engineering",
      "reportCount": 156
    },
    {
      "userId": "user-uuid-2",
      "email": "bob@company.com",
      "displayName": "Bob Smith",
      "clusterId": "cluster-id-456",
      "clusterName": "Operations",
      "reportCount": 142
    }
  ],
  "bottomUsers": [
    {
      "userId": "user-uuid-100",
      "email": "zoe@company.com",
      "displayName": "Zoe Davis",
      "clusterId": "cluster-id-789",
      "clusterName": "Sales",
      "reportCount": 1
    }
  ],
  "limit": 10,
  "totalUsers": 245
}
```

**Data Source**:
- Primary: `lamiUserWorkReport` table (count by userID)
- Secondary: `allLamiUsers` table (user details)
- Filter: Cluster access control

**Response Time**: <5 seconds

**Notes**:
- `bottomUsers` array is reversed so highest count is first
- `reportCount` is cumulative across all time
- Users with 0 reports may not appear in results

---

### Endpoint 3: GET /management/stats/usage/coverage-metrics

**Authorization**: 
- Requires `requesterEmail` parameter
- User must have `can_manage_reports` OR `can_manage_users` permission

**URL Format**:
```
GET /management/stats/usage/coverage-metrics?requesterEmail=user@domain.com
```

**Query Parameters**:
```
requesterEmail: string (EmailStr) - REQUIRED - User's email address
```

**Response Status Codes**:
- `200 OK` - Successfully retrieved metrics
- `403 Forbidden` - User lacks permission
- `500 Internal Server Error` - DynamoDB error

**Response Body (200 OK)**:
```json
{
  "totalUsers": 245,
  "usersWithReports": 198,
  "overallUsagePercentage": 80.82,
  "coverageTrend": "improved",
  "totalReports": 24567,
  "monthlyTrend": [
    {"month": "2024-02", "reportCount": 1850},
    {"month": "2024-03", "reportCount": 1920},
    {"month": "2024-04", "reportCount": 2045},
    {"month": "2024-05", "reportCount": 2118}
  ],
  "clusterCoverage": [
    {
      "clusterId": "cluster-id-123",
      "clusterName": "Engineering",
      "totalUsers": 80,
      "activeUsers": 73,
      "usagePercentage": 91.25
    },
    {
      "clusterId": "cluster-id-456",
      "clusterName": "Operations",
      "totalUsers": 95,
      "activeUsers": 82,
      "usagePercentage": 86.32
    },
    {
      "clusterId": "cluster-id-789",
      "clusterName": "Sales",
      "totalUsers": 70,
      "activeUsers": 43,
      "usagePercentage": 61.43
    }
  ]
}
```

**Coverage Trend Interpretation**:
```
"improved"  - Last month has >10% more reports than previous month
"decreased" - Last month has <90% of previous month's reports
"neutral"   - Within ±10% of previous month
```

**Data Source**:
- Primary: `lamiUserWorkReport` table
- Secondary: `allLamiUsers` table
- Filter: Cluster access control

**Response Time**: <10 seconds

**Monthly Trend**:
- Returns last 12 months of data
- Format: "YYYY-MM"
- Includes months with 0 reports
- Sorted chronologically

---

## Component API

### WorkReportUsageStats.js

**Props**:
```javascript
{
  apiBaseUrl: string,           // Base URL for API calls (e.g., "http://localhost:8000")
  requesterEmail: string,        // User's email for authentication
  open: boolean,                 // Whether the parent modal is open
  profile: {                      // User profile data
    isSuperuser: boolean,
    canManageUsers: boolean,
    canManageReports: boolean,
    leaderClusterIds: string[],
    clusterIds: string[]
  }
}
```

**Methods** (Internal - exposed for testing):
- `buildUrl(base, path, params)` - Constructs API URLs with query params
- `fetchInactiveUsers()` - Calls GET /management/stats/usage/inactive-users
- `fetchTopBottomUsers()` - Calls GET /management/stats/usage/top-bottom-users
- `fetchCoverageMetrics()` - Calls GET /management/stats/usage/coverage-metrics

**State Variables** (Internal):
```javascript
activeView: 'inactive' | 'topbottom' | 'coverage'
inactiveUsers: Array<UserRecord>
topBottomUsers: { topUsers: Array, bottomUsers: Array, limit, totalUsers }
coverageMetrics: { totalUsers, usersWithReports, ... (see response format above) }
loadingInactive: boolean
loadingTopBottom: boolean
loadingCoverage: boolean
errorInactive: string | null
errorTopBottom: string | null
errorCoverage: string | null
inactiveDays: number (1-365)
topBottomLimit: number (1-50)
```

**Styling**:
- All styles defined inline in `styles` object
- Uses standard CSS properties
- No external CSS dependencies
- Responsive grid layouts
- Mobile-friendly

---

## Database Schema Details

### Table: lamiUserWorkReport
**Relevant Fields**:
```
reportID (PK): string - Unique report identifier
userID (or userId): string - User who created report
reportDate: string - ISO 8601 timestamp
```

**Scan Filter**:
```python
# Get all reports
all_reports = scan_table_items(work_report_table)

# For each report, extract userID and reportDate
for report in all_reports:
    user_id = report.get('userID') or report.get('userId')
    report_date = report.get('reportDate')
```

### Table: allLamiUsers
**Relevant Fields**:
```
id (PK): string - User ID
email: string - User's email address
displayName: string - For display in UI
clusterID (or clusterId): string - User's assigned cluster
NAME(ENGLISH) or nameEnglish: string - User's name
```

**Lookup**:
```python
# Get user details by ID
user_record = all_users_table.get_item(Key={'id': user_id})
```

### Table: lamiCluster
**Relevant Fields**:
```
id or clusterID (PK): string - Cluster identifier
name: string - Cluster display name
```

---

## Error Handling

### Common Error Scenarios

**1. User Not Authorized**
```
Status: 403 Forbidden
Response: {"detail": "You do not have permission to view usage statistics."}
Cause: User lacks can_manage_reports or can_manage_users permission
```

**2. DynamoDB Error**
```
Status: 500 Internal Server Error
Response: {"detail": "Failed to load inactive users."}
Cause: DynamoDB scan failed (connectivity, permissions, table issues)
```

**3. Invalid Parameters**
```
Status: 400 Bad Request (would be returned if validation added)
Response: {"detail": "Invalid days parameter"}
Cause: Parameter out of valid range
```

**4. Network Error (Frontend)**
```
Error message displayed: "Failed to load [view name]."
Cause: Network connectivity issue or fetch failed
Recovery: User can retry with "Load Data" button
```

---

## Performance Considerations

### Current Approach
- Full table scan of `lamiUserWorkReport`
- In-memory filtering and aggregation
- Suitable for teams up to ~10,000 users

### Scaling Recommendations

**For 10,000+ users**:
1. Add DynamoDB Global Secondary Index (GSI) on `reportDate`
2. Add DynamoDB GSI on `userID`
3. Implement pagination in endpoints
4. Add results caching (Redis/DynamoDB TTL)

**Query Optimization Example**:
```python
# Instead of scanning all, use GSI
response = work_report_table.query(
    IndexName='reportDate-index',
    KeyConditionExpression='reportDate > :cutoff_date',
    ExpressionAttributeValues={
        ':cutoff_date': cutoff_date_iso
    }
)
```

### Current Response Times

| Endpoint | Data Size | Response Time |
|----------|-----------|--------------|
| inactive-users | <1000 users | <1 sec |
| top-bottom-users | <5000 users | <3 sec |
| coverage-metrics | all data | <5 sec |

---

## Code Structure

### Backend (main.py)

**Location of endpoints**: ~Line 2874 in main.py

**Helper Functions Used**:
- `build_management_context()` - Validates user permissions
- `load_clusters()` - Loads all clusters
- `scan_table_items()` - Scans DynamoDB table
- `dynamo_to_python()` - Converts DynamoDB types
- `cluster_id_from_record()` - Extracts cluster ID

**Import Dependencies**:
```python
from datetime import datetime, timedelta
from boto3.dynamodb.conditions import Attr
from typing import Dict, List, Optional, Any
```

### Frontend (React)

**File**: `lami-report-web/components/WorkReportUsageStats.js`
**Component Type**: Functional React component
**Hooks Used**: 
- `useState` - For state management
- `useCallback` - For memoized callbacks
- `useMemo` - For computed values

**Child Components**: None (self-contained)

**Integration Point**: 
- Imported in ManagementModal.js
- Rendered via `renderUsageStatsView()` function
- Controlled by `statsSubTab` state

---

## Deployment Checklist

- [ ] Backend: Copy new endpoint code into main.py
- [ ] Backend: Test endpoints with curl/Postman
- [ ] Backend: Verify DynamoDB permissions
- [ ] Frontend: Copy WorkReportUsageStats.js to components/
- [ ] Frontend: Update ManagementModal.js with import
- [ ] Frontend: Test sub-tab switching
- [ ] Frontend: Test data loading in each view
- [ ] Frontend: Test error states
- [ ] Integration: Test end-to-end as superuser
- [ ] Integration: Test end-to-end as cluster leader
- [ ] Production: Monitor CloudWatch logs for errors
- [ ] Production: Verify response times acceptable

---

## Testing Guide

### Unit Test Examples

**Backend - Test inactive users**:
```python
def test_inactive_users_endpoint():
    response = client.get(
        "/management/stats/usage/inactive-users",
        params={
            "requesterEmail": "test@company.com",
            "days": 30
        }
    )
    assert response.status_code == 200
    assert "totalInactiveUsers" in response.json()
    assert "users" in response.json()
```

**Frontend - Test view switching**:
```javascript
test('switches views when tab clicked', () => {
    const {getByText} = render(
        <WorkReportUsageStats 
            apiBaseUrl="http://localhost"
            requesterEmail="test@company.com"
            open={true}
            profile={{canManageReports: true}}
        />
    );
    const button = getByText('Coverage Metrics');
    fireEvent.click(button);
    expect(getByText('Overall Usage Percentage')).toBeInTheDocument();
});
```

### Integration Test Scenarios

1. **Happy Path**: 
   - User accesses dashboard
   - Loads all three views
   - Data displays correctly

2. **Permission Denied**:
   - Regular user tries to access
   - Gets 403 error
   - Graceful error message

3. **No Data**:
   - New system with no reports
   - Shows empty states
   - No errors

4. **Large Dataset**:
   - 10,000+ users
   - Endpoints still respond
   - Performance acceptable

---

## Monitoring & Logs

### CloudWatch Metrics to Monitor
```
- Duration of DynamoDB scans
- 403 Forbidden errors (permission issues)
- 500 errors (DynamoDB failures)
- Response times per endpoint
```

### Log Statements Added
```python
logger.info(f"Fetching inactive users with threshold {days} days")
logger.error("Failed to get inactive users.", exc_info=True)
logger.info(f"Found {len(results)} inactive users")
```

### Frontend Error Logging
```javascript
console.error("Failed to load inactive users:", error);
// Displayed to user as: "Failed to load inactive users."
```

---

## Maintenance Guide

### Common Issues & Solutions

**Issue**: Data seems stale
- **Solution**: Click "Load Data" button to refresh (no cache)

**Issue**: Wrong cluster data showing
- **Solution**: Verify user's leaderClusterIds in allLamiUsers

**Issue**: Performance degradation
- **Solution**: Check DynamoDB throughput; add indexes if in production

**Issue**: Users missing from results
- **Solution**: Verify they have reportID in lamiUserWorkReport

---

## Future Enhancements

### Planned Features (Priority Order)

1. **CSV Export** (Low effort, high value)
   - Export each view as CSV file
   - Add export button per view

2. **Custom Date Range** (Medium effort)
   - Allow users to select date range
   - Affects all calculations

3. **Cluster Comparison** (Medium effort)
   - Side-by-side cluster metrics
   - Competitive benchmarking view

4. **Drill-Down Details** (Medium effort)
   - Click user to see their report history
   - Click cluster for detailed breakdown

5. **Caching Layer** (High effort, enables scale)
   - Redis caching of metrics
   - TTL-based refresh
   - Significant performance improvement

6. **Scheduled Reports** (High effort)
   - Automated weekly/monthly email
   - Stakeholder notifications
   - Trend analysis included

---

## References & Documentation

- **API Client Code**: See `buildUrl()` and `fetch()` calls in WorkReportUsageStats.js
- **Permission Checks**: See `build_management_context()` in main.py
- **DynamoDB Queries**: See `scan_table_items()` and filtering logic
- **Existing Stats Implementation**: See `renderWorkStatsView()` for patterns

---

## Version Information

- **Created**: February 2026
- **Backend Framework**: FastAPI
- **Frontend Framework**: React
- **Python Version**: 3.7+
- **Node.js Version**: 14+

---

This technical specification should enable developers to:
- Maintain the current implementation
- Add new features
- Debug issues
- Scale for future growth
- Integrate with other systems
