# LamiPak Report Generation System

A comprehensive, multi-platform reporting system for generating, managing, and analyzing work reports and technical assessments. The system includes mobile apps (iOS/Android), a web dashboard, and a powerful backend API with analytics capabilities.

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Repositories & Tech Stack](#repositories--tech-stack)
- [Repository Details](#repository-details)
- [Codebase Structure & Architecture](#codebase-structure--architecture)
- [Deployment & Environments](#deployment--environments)
- [Integrations & External Services](#integrations--external-services)
- [Access & Ownership](#access--ownership)
- [Quick Start](#quick-start)
- [Development](#development)

---

## 🎯 Project Overview

LamIPak Report Generation System is an enterprise-grade reporting platform that enables teams to:
- Create and manage detailed work reports and technical assessments
- Track usage metrics and team adoption
- Generate analytics dashboards for cluster heads and managers
- Support multiple languages (English, Chinese)
- Provide role-based access control and permissions
- Store all data in AWS with secure authentication

The system consists of three main applications:
1. **Work Report Web** - React-based web dashboard for work report generation
2. **LamIPak Report Mobile** - React Native mobile app (iOS/Android) for detailed technical assessments
3. **Report Backend** - FastAPI server handling report generation, analytics, and data storage

---

## 🏗️ Repositories & Tech Stack

| Repository | Purpose | Tech Stack | Platform |
|------------|---------|-----------|----------|
| **lami-report-backend** | REST API for report generation, KPI stats, management endpoints | Python 3.11, FastAPI, Uvicorn | Server (Docker, Koyeb) |
| **lami-report-web** | Web dashboard for work report generation and submission | React 18.3, Expo 52, React Native Web | Web (Browser) |
| **lami-report-mobile** | Mobile app for comprehensive technical report creation | React Native 0.74, Expo 51, React Navigation | iOS/Android (Expo) |

### Key Technology Choices

**Backend:**
- **Framework**: FastAPI (async, high-performance REST API)
- **Language**: Python 3.11 with type hints
- **Process Manager**: Supervisord with Uvicorn
- **Containerization**: Docker with multi-service orchestration
- **Translation**: LibreTranslate (self-hosted) for multi-language support

**Frontend (Web & Mobile):**
- **Framework**: React 18.3 with React Native
- **Build Tool**: Expo for unified iOS, Android, and Web builds
- **Navigation**: React Navigation (drawer and tab-based navigation)
- **UI Components**: React Native Paper (Material Design)
- **State Management**: Context API + custom hooks
- **Localization**: i18n-js for multi-language support

**Authentication & Cloud:**
- **Auth**: AWS Cognito via AWS Amplify
- **Database**: AWS DynamoDB (primary data storage)
- **Storage**: AWS S3 (file uploads, exports)
- **File Processing**: LibreOffice (document conversion), Pillow (image processing)

---

## 📦 Repository Details

### 1. **lami-report-backend**
Backend API server providing report generation, analytics, and management capabilities.

**Purpose & Scope:**
- Generate downloadable work reports in pdf format
- Provide real-time KPI statistics and usage analytics
- Manage user profiles and cluster-level access control
- Handle multi-language translation requests
- Support report filtering, export, and archival

**Tech Stack Details:**
- Framework: FastAPI with CORS middleware
- Database: AWS DynamoDB (multiple tables: lamiUserWorkReport, allLamiUsers, lamiTechnicalReport, etc.)
- File Processing: docxtpl, Pillow, LibreOffice, docxcompose
- Analytics: NumPy, Matplotlib, Seaborn
- Translation: LibreTranslate (self-hosted)
- AWS SDK: Boto3

**Key Dependencies:**
```
Core: fastapi, uvicorn, pydantic
AWS: boto3, botocore
Database: dynamodb-conditions
File Processing: docxtpl, docxcompose, pillow
Data Science: numpy, matplotlib, seaborn
Translation: libretranslate, argostranslate
Utilities: email-validator, python-dateutil, pytz
```

**Main Endpoints:**
- `GET/POST /reports` - Retrieve and create reports
- `POST /reports/generate` - Generate downloadable DOCX reports
- `GET /management/stats/usage/*` - Usage statistics endpoints
- `GET /management/profile` - User profile and permissions
- `POST /translate` - Text translation service

---

### 2. **lami-report-web**
Web-based dashboard for work report creation and management.

**Purpose & Scope:**
- User authentication via AWS Cognito
- Work report creation and submission interface
- View submitted reports and analytics
- Management modal for cluster administrators
- Multi-language support (English, Chinese, System Default)
- Real-time API communication with backend

**Tech Stack Details:**
- Framework: React 18.3 with Hooks
- Build: Expo 52 (unified build for web, iOS, Android)
- Authentication: AWS Amplify (@aws-amplify/ui-react)
- Navigation: Bottom tab navigation (React Navigation)
- State Management: React Context API
- Async Storage: @react-native-async-storage for local data caching
- UI Library: React Native Paper for web styling

**Key Dependencies:**
```
Core: react, react-dom, react-native
Build: expo, @expo/metro-runtime
AWS: aws-amplify, @aws-amplify/ui-react
Navigation: @react-navigation/bottom-tabs, @react-navigation/native
Storage: @react-native-async-storage/async-storage
Date Handling: date-fns
Localization: i18n-js
UI: react-native-paper, react-native-vector-icons
```

**Main Features:**
- User authentication dashboard
- Work report form builder
- Management modal with:
  - Usage metrics (inactive users, top/bottom performers)
  - Coverage metrics and trends
  - User and report management
- Language selector with system default option
- Responsive web design with fixed top navigation

---

### 3. **lami-report-mobile**
Native mobile application (iOS/Android) for comprehensive technical report generation.

**Purpose & Scope:**
- Detailed technical assessment form workflow
- Multi-screen report builder with context preservation
- Draft auto-save and restoration
- Local SQLite database for offline access
- Image capture and file attachment
- Real-time KPI statistics
- Management and analytics screens
- Supports work reports and technical reports

**Tech Stack Details:**
- Framework: React Native 0.74 with Expo 51
- Build System: EAS (Expo Application Services)
- Database: Expo SQLite for local storage
- Navigation: React Navigation (drawer + tabs)
- UI Framework: React Native Paper
- State Management: Context API with custom providers
- File System: Expo FileSystem API
- Image Processing: expo-image-picker
- Authentication: AWS Amplify (React Native)

**Key Dependencies:**
```
Core: react-native, expo
AWS: aws-amplify, @aws-amplify/react-native, @aws-amplify/ui-react-native
Navigation: @react-navigation/drawer, @react-navigation/stack, @react-navigation/material-top-tabs
Database: expo-sqlite
File System: expo-file-system, react-native-file-viewer, react-native-fs
UI: react-native-paper, react-native-gesture-handler
Utilities: date-fns, moment, i18n-js
```

**Build Configuration (EAS):**
- **Development**: Development client with internal distribution
- **Preview**: APK builds for Android, environment variables passed via EAS
- **Production**: iOS via macos-sonoma with Xcode 16.0

**App Store Details:**
- iOS App ID: 6741105334
- Deployment: App Store Connect

---

## 🏛️ Codebase Structure & Architecture

### Backend Architecture

```
lami-report-backend/
├── app/
│   ├── main.py                  # FastAPI app setup, all endpoints
│   ├── get_kpi_stats.py        # KPI stats calculation and workbook generation
│   └── build_sample_workbook.py # Sample data generation
├── db/
│   └── sessions/               # Session management data
├── scripts/                     # Utility scripts
├── Dockerfile                  # Docker containerization
├── supervisord.conf           # Process manager config
├── requirements.txt           # Python dependencies
├── export_dynamo_schema.py    # Schema export utilities
└── cleanupDB.py               # Database maintenance
```

**Key Architecture Patterns:**

1. **Microservices with Supervisord:**
   - FastAPI server (uvicorn on port 8000)
   - LibreTranslate service (port 5000)
   - Both managed by supervisord for reliability

2. **Data Flow:**
   ```
   Mobile/Web Client → AWS Cognito (Auth) → FastAPI Server → DynamoDB
                                         → LibreTranslate (Translation)
                                         → S3 (File Storage)
   ```

3. **Report Generation Pipeline:**
   - User submits form data
   - Backend receives request with authentication
   - Validates user permissions via DynamoDB
   - Fetches KPI stats and historical data
   - Generates DOCX using docxtpl templates
   - Returns file to client for download

4. **Database Schema:**
   - `allLamiUsers` - User profiles and permissions
   - `lamiUserWorkReport` - Work report submissions
   - `lamiTechnicalReport` - Technical assessment data
   - `lamiTechnicalReportDrafts` - Draft storage
   - `lamiCluster` - Organization structure

---

### Web App Architecture

```
lami-report-web/
├── App.js                   # Main app component, theme setup
├── index.js                 # Entry point
├── UserContext.js          # User state management
├── components/
│   ├── WorkReportScreen.js     # Main work report form
│   ├── ManagementModal.js      # Admin panel with stats
│   └── WorkReportUsageStats.js # Usage metrics dashboard
├── contexts/
│   ├── LanguageContext.js   # Language/i18n management
│   ├── UnsavedChangesContext.js
│   └── Other context providers
├── i18n/
│   ├── i18n.js             # i18n configuration
│   └── translations.js     # Translation strings
├── src/
│   ├── aws-exports.js      # AWS Amplify config
│   └── amplifyconfiguration.json
└── assets/                 # Images, icons, etc.
```

**Architectural Patterns:**

1. **Authentication Flow:**
   ```
   Unauthenticated → Amplify Authenticator
                  → AWS Cognito Sign-in
                  → Authenticated App
   ```

2. **API Integration:**
   - Runtime environment detection (localhost vs production)
   - API base URL: `https://lamipak.koyeb.app/` (production)
   - Local fallback: `http://localhost:8000/` (development)
   - Query parameter building with proper encoding

3. **State Management:**
   - Language Context: Global language selection
   - User Context: Authenticated user information
   - Component-level state for forms and modals

4. **Top Navigation Bar:**
   - Fixed position (z-index: 1000)
   - Color: #6600cc (purple)
   - Actions: Language selector, Manage button, Sign out

---

### Mobile App Architecture

```
lami-report-mobile/
├── App.js                          # Main app entry, navigation setup
├── ReportContext.js               # Report state management
├── UserContext.js                 # User/auth state
├── components/
│   ├── TechnicalReportScreen.js   # Technical report form
│   ├── WorkReportScreen.js        # Work report form
│   ├── ExecutiveSummary.js
│   ├── GenerateReport.js          # Report generation
│   └── [20+ other report components]
├── contexts/
│   ├── TechnicalDraftContext.js   # Draft persistence
│   ├── WorkDraftContext.js
│   ├── ClusterContext.js          # Cluster selection
│   ├── WorkReportFiltersContext.js
│   ├── UnsavedChangesContext.js
│   └── LanguageContext.js
├── utils/
│   ├── confirmAction.js           # Action confirmations
│   ├── reportDefaults.js          # Default values
│   └── useTechnicalHeaderButtons.js
├── android/                       # Android-specific configs
├── i18n/
│   ├── i18n.js
│   └── translations.js
├── src/
│   ├── aws-exports.js
│   └── amplifyconfiguration.json
├── amplify/                       # AWS Amplify configuration
├── eas.json                       # EAS build configuration
└── package.json
```

**Navigation Structure:**

```
Drawer Navigator (Main)
├── ReportInfo Screen
├── MachineDetails Screen
├── PackageDetails Screen
├── ExecutiveSummary Screen
├── Background Screen
├── ActionsTaken Screen
├── Conclusion Screen
├── Proposal Screen
└── GenerateReport Screen

Tab Navigator (Alternative)
├── Work Reports Tab
├── Technical Reports Tab
├── Stats Tab
└── Management Tab
```

---

## 🚀 Deployment & Environments

### Development Environment

**Local Development Setup:**

```bash
# Backend
cd lami-report-backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
export ENABLE_USAGE_METRICS=true
python -m uvicorn app.main:app --reload --port 8000

# Frontend (Web)
cd lami-report-web
npm install
npm start  # Opens Expo dev server

# Mobile (lami-report-mobile)
cd lami-report-mobile
npm install
npm start
# Then select 's' for web, 'a' for Android, or 'i' for iOS
```

**Environment Variables (.env):**
```
# Backend
AWS_ACCESS_KEY_ID=<aws-key>
AWS_SECRET_ACCESS_KEY=<aws-secret>
AWS_REGION=us-east-1
ENABLE_USAGE_METRICS=true
EXPO_PUBLIC_API_BASE_URL=http://localhost:8000/

# Frontend
EXPO_PUBLIC_API_BASE_URL=http://localhost:8000/
EXPO_PUBLIC_SPOOF_EMAIL=  # Optional: for local role testing
```

---

### Staging Environment

**Hosted Platform:** Koyeb (Docker-based PaaS)

**Deployment Configuration:**
- **API Base URL:** `https://lamipak.koyeb.app/`
- **Container:** Python 3.11-slim with LibreOffice
- **Processes:**
  - FastAPI server (port 8000)
  - LibreTranslate service (port 5000)
- **Database:** AWS DynamoDB (shared with production)
- **Authentication:** AWS Cognito (production instance)

---

### Production Environment

**Web & Mobile:**
- **Web**: Deployed via Expo, served through web browsers
- **iOS**: App Store (App ID: 6741105334)
- **Android**: APK file (via Expo)
- **API Endpoint**: `https://lamipak.koyeb.app/`

**Backend:**
- **Hosting**: Koyeb (Docker container)
- **Compute**: Auto-scaling based on load
- **Database**: AWS DynamoDB (production region: us-east-1)
- **Storage**: AWS S3 for file exports
- **Monitoring**: CloudWatch logs and metrics

**Build Pipeline:**

```
Mobile Apps:
├── Development Build
│   └── Expo Development Client (internal)
├── Preview Build
│   └── EAS Build → APK/IPA (testing)
└── Production Build
    └── EAS Build → App Store/Play Store

Backend:
└── Docker Image
    └── Koyeb Container
        └── Supervisord manages FastAPI + LibreTranslate
```

## 🔌 Integrations & External Services

### AWS Services

| Service | Purpose | Usage |
|---------|---------|-------|
| **Cognito** | User authentication and authorization | Sign-in, role management, JWT tokens |
| **DynamoDB** | Primary database | User data, reports, KPI stats, drafts |
| **S3** | File storage | Report exports, user uploads, backups |
| **Lambda** | Serverless compute (optional) | Scheduled tasks, event processing |
| **CloudWatch** | Monitoring and logging | Application logs, performance metrics, alerts |
| **API Gateway** | API management (optional) | Rate limiting, request validation, analytics |

### Third-Party Services

| Service | Purpose | Integration |
|---------|---------|-------------|
| **LibreTranslate** | Multi-language translation | Self-hosted on Koyeb, supports EN/ZH |
| **Koyeb** | Container hosting | FastAPI backend deployment |
| **Expo** | Mobile app build/distribution | iOS/Android build pipeline, EAS |
| **App Store Connect** | iOS distribution | Publish to Apple App Store |
| **Google Play Console** | Android distribution | Publish to Google Play Store |
| **LibreOffice** | Document conversion | DOCX generation, formatting |

### Internal APIs

**Backend API Endpoints:**

```
Authentication & Profiles:
POST   /management/login
GET    /management/profile?requesterEmail=<email>

Reports:
GET    /reports?userID=<id>&reportID=<id>
POST   /reports - Submit new report
POST   /reports/generate - Generate downloadable DOCX
GET    /reports/<reportID> - Retrieve specific report

KPI Statistics:
GET    /management/stats/kpi?...
POST   /management/stats/kpi:export

Usage Statistics (Feature-flagged):
GET    /management/stats/usage/inactive-users?requesterEmail=<email>&days=30
GET    /management/stats/usage/top-bottom-users?requesterEmail=<email>&limit=10
GET    /management/stats/usage/coverage-metrics?requesterEmail=<email>

Translation:
POST   /translate - Translate text EN ↔ ZH

Admin Management:
GET/POST /management/users
GET/POST /management/clusters
DELETE   /management/users/<userID>
```

---

## 👥 Access & Ownership

### AWS Access & Ownership

| Resource | Owner | Access |
|----------|-------|--------|
| **AWS Account** | Pradeep Tyagi | Root account controlled by Pradeep Tyagi |
| **DynamoDB** | Pradeep Tyagi | Read/Write via IAM roles controlled by Pradeep Tyagi |
| **S3 Buckets** | Pradeep Tyagi | Upload/Download via IAM policies controlled by Pradeep Tyagi |
| **Cognito User Pools** | Pradeep Tyagi | User management via AWS console controlled by Pradeep Tyagi |
| **CloudWatch** | Pradeep Tyagi | Log viewing and alerts controlled by Pradeep Tyagi |
| **Lambda Functions** | Pradeep Tyagi | Code deployment via CI/CD controlled by Pradeep Tyagi |

---

### Repository Access

| Repository | Owner | Access Level |
|------------|-------|--------------|
| **lami-report-backend** | Pradeep Tyagi | Admin |
| **lami-report-web** | Pradeep Tyagi | Admin |
| **lami-report-mobile** | Pradeep Tyagi | Admin |

---

### Deployment Platform Access

| Platform | Services | Owners | Credentials |
|----------|----------|--------|-------------|
| **Koyeb** | Backend hosting, LibreTranslate | Pradeep Tyagi | Dashboard login controlled by Pradeep Tyagi |
| **Expo** | Build pipeline, distribution | Pradeep Tyagi | Expo account + EAS CLI controlled by Pradeep Tyagi |
| **App Store Connect** | iOS app distribution | Pradeep Tyagi | Apple Developer account controlled by Pradeep Tyagi |
| **Google Play Console** | Android app distribution | Pradeep Tyagi | Google Play Developer account controlled by Pradeep Tyagi |
| **DockerHub** | Container images | Pradeep Tyagi | registry.hub.docker.com credentials controlled by Pradeep Tyagi |

---

### Service Credentials Management

**Secure Storage:**
- AWS credentials: AWS IAM roles (no hardcoded keys)
- Cognito config: Repository files (public)
- Database encryption: AWS KMS (enabled by default)
- API keys: Koyeb secrets

**Rotation Policy:**
- AWS tokens: Automatically rotated via IAM
- App Store credentials: Renewed annually
- Docker registry: Update on request
- Database backups: Daily (AWS automated)

---

## 🎬 Quick Start

### Prerequisites
- Node.js 16+ and npm/yarn
- Python 3.11+
- AWS account with DynamoDB, S3, and Cognito configured
- Docker (for backend deployment)

### 1. Clone Repositories

```bash
# Individual repos
git clone https://github.com/pradeeptyagi23/lami-report-backend.git lami-report-backend
git clone https://github.com/pradeeptyagi23/lami-report-web.git lami-report-web
git clone https://github.com/pradeeptyagi23/lami-report-mobile.git lami-report-mobile
```

### 2. Backend Setup

```bash
cd lami-report-backend

# Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Configure AWS
export AWS_REGION=us-east-1
export AWS_ACCESS_KEY_ID=your-key-id
export AWS_SECRET_ACCESS_KEY=your-secret-key

# Set feature flags
export ENABLE_USAGE_METRICS=true

# Run server
python -m uvicorn app.main:app --reload --port 8000
```

### 3. Web App Setup

```bash
cd lami-report-web

# Install dependencies
npm install

# Create .env file
echo 'EXPO_PUBLIC_API_BASE_URL=http://localhost:8000/' > .env

# Start development server
npx expo start -w

# Select 'w' for web browser
```

### 4. Mobile App Setup

```bash
cd lami-report-mobile

# Install dependencies
npm install

# Create .env file
echo 'EXPO_PUBLIC_API_BASE_URL=http://localhost:8000/' > .env

# Start development server
npm start

# Select 'a' for Android or 'i' for iOS
```

### 5. AWS Configuration

Ensure the following AWS services are configured:

```yaml
Cognito:
  - User pool created
  - App client configured
  - User groups for permissions (admin, manager, user)

DynamoDB:
  - Tables created: lamiUser, allLamiUsers, lamiUserWorkReport, lamiTechnicalReport
  - Global secondary indexes configured
  - Backups enabled

S3:
  - lamiusertechreports and lamiuserworkreports Bucket created for file uploads
  - Lifecycle policies for cleanup

```

---

## 💻 Development

### Code Organization

- **Backend**: Modular FastAPI with service-oriented architecture
- **Frontend**: Component-based React with context API
- **Mobile**: Navigation-based screens with context providers

### Key Development Commands

```bash
# Backend
pip install -r requirements.txt  # Install dependencies
python -m pytest tests/          # Run unit tests
python export_dynamo_schema.py   # Export DB schema
python cleanupDB.py              # Cleanup old records

# Frontend/Mobile
npm install                      # Install dependencies
npm start                        # Start Expo dev server
npm run web                      # Run web version
npm run android                  # Run Android version
npm run ios                      # Run iOS version
```

### Development Features

- **Hot reload**: Both backend and frontend support hot module replacement
- **Feature flags**: `ENABLE_USAGE_METRICS` to toggle analytics
- **Spoof email**: `EXPO_PUBLIC_SPOOF_EMAIL` for role testing locally
- **Debug logging**: Console logs for API requests/responses
- **Draft auto-save**: Technical Assistant saves form state to SQLite


---

## 📊 Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                       External Users                            │
└────────────┬──────────────────────────────────────────────┬─────┘
             │                                              │
    ┌────────▼──────┐                          ┌──────────▼────┐
    │ Web Browser   │                          │  Mobile App   │
    │ (lami-web)    │                          │ (iOS/Android) │
    └────────┬──────┘                          └──────────┬────┘
             │                                           │
    ┌────────▼────────────────────────────────┬─────────▼──┐
    │                                         │            │
┌───▼──────────────────────────────────────┐ │            │
│    AWS Cognito (Authentication)          │ │            │
└───┬──────────────────────────────────────┘ │            │
    │                                        │            │
    └────────────────────┬────────────────────┼───────────┘
                         │ JWT Token         │
                    ┌────▼─────────────────┐ │
                    │                      │ │
        ┌───────────▼────────────────────┘ │
        │ FastAPI Backend (Koyeb)         │
        │ - Report generation              │
        │ - KPI statistics                 │
        │ - Management APIs                │
        ├────────────────────────┬─────────┘
        │                        │
        │  ┌──────────────────┐  │
        │  │  LibreTranslate  │  │
        │  │   (Port 5000)    │  │
        │  └──────────────────┘  │
        │                        │
        └───┬────────────────────┘
            │
       ┌────▼─────────────────────────────┐
       │    AWS Services                  │
       ├──────────────────────────────────┤
       │ • DynamoDB (Data Storage)        │
       │ • S3 (File Storage)              │
       │ • CloudWatch (Logging)           │
       │ • Lambda (Optional Tasks)        │
       └──────────────────────────────────┘
```




---

## 📞 Support & Contact

Pradeep Tyagi
Email - pradeep.tyagi@lamipak.biz

---

