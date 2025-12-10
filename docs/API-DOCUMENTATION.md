# API Documentation for Pulse 360° Admin Portal

**Last Updated:** November 17, 2025  
**Version:** 1.0

## Table of Contents

1. [Overview](#overview)
2. [Authentication Architecture](#authentication-architecture)
3. [Environment Configuration](#environment-configuration)
4. [Microsoft Graph API](#microsoft-graph-api)
5. [Azure Resource Manager API](#azure-resource-manager-api)
6. [Power Platform API](#power-platform-api)
7. [Dataverse API](#dataverse-api)
8. [Microsoft Entra ID (Azure AD)](#microsoft-entra-id-azure-ad)
9. [Application Insights API](#application-insights-api)
10. [Log Analytics API](#log-analytics-api)
11. [Internal API Routes](#internal-api-routes)
12. [Microsoft Defender for Endpoint API](#microsoft-defender-for-endpoint-api)
13. [Microsoft Graph Reports API](#microsoft-graph-reports-api)
14. [Windows Update State API (Intune)](#windows-update-state-api-intune)
15. [ServiceNow Integration API](#servicenow-integration-api)
16. [Required Permissions Summary](#required-permissions-summary)
17. [Troubleshooting](#troubleshooting)

---

## Overview

Pulse 360° is a Next.js application that integrates with multiple Microsoft cloud services to provide a unified administration portal for:

- **Microsoft 365** (via Microsoft Graph)
- **Azure** (via Azure Resource Manager)
- **Power Platform** (via Power Platform API and Dataverse)
- **Microsoft Entra ID** (formerly Azure AD)
- **Security & Compliance** (Defender, Sentinel, Intune)

This document provides IT administrators with detailed information about all external API endpoints, authentication mechanisms, required permissions, and configuration steps.

---

## Authentication Architecture

### Authentication Flow

Pulse 360° uses **NextAuth.js** with the Azure AD provider for user authentication and **MSAL (Microsoft Authentication Library)** for server-side token acquisition.

#### Flow Diagram:

```
User Browser
    ↓ (1) Login request
NextAuth.js (Frontend)
    ↓ (2) OAuth redirect to Microsoft Entra ID
Microsoft Entra ID
    ↓ (3) User authenticates
    ↓ (4) Returns authorization code
NextAuth.js
    ↓ (5) Exchanges code for tokens (access, id, refresh)
    ↓ (6) Stores tokens in JWT session
API Routes (Server)
    ↓ (7) Uses tokens to acquire resource-specific tokens via:
    ├── On-Behalf-Of (OBO) flow
    ├── Refresh token exchange
    └── Client credentials flow
External APIs (Graph, ARM, Power Platform)
```

### Token Types

1. **Access Token**: Short-lived token for user-delegated API access
2. **ID Token**: Contains user identity claims
3. **Refresh Token**: Long-lived token for acquiring new access tokens
4. **Resource-Specific Tokens**: Acquired via OBO or client credentials for specific APIs

---

## Environment Configuration

### Required Environment Variables

Create a `.env.local` file in the project root with the following variables:

```bash
# ============================================
# NEXTAUTH CONFIGURATION
# ============================================
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=<random-secret-string>
# Generate with: openssl rand -base64 32

# ============================================
# MICROSOFT ENTRA ID (AZURE AD) APP REGISTRATION
# ============================================
AZURE_CLIENT_ID=<your-app-registration-client-id>
AZURE_CLIENT_SECRET=<your-app-registration-client-secret>
AZURE_TENANT_ID=<your-tenant-id>

# Alternative environment variable names (NextAuth v5 compatibility)
AUTH_AZURE_AD_CLIENT_ID=<your-app-registration-client-id>
AUTH_AZURE_AD_CLIENT_SECRET=<your-app-registration-client-secret>
AUTH_AZURE_AD_TENANT_ID=<your-tenant-id>
AUTH_SECRET=<random-secret-string>

# ============================================
# AZURE SUBSCRIPTION (Optional - for Azure Resource Graph)
# ============================================
AZURE_SUBSCRIPTION_ID=<your-subscription-id>

# ============================================
# OPTIONAL: DEFENDER FOR CLOUD
# ============================================
DEFENDER_MANAGEMENT_SCOPE=https://management.azure.com/.default
```

### Environment Variable Locations

| Variable | Storage Location | Purpose |
|----------|------------------|---------|
| `NEXTAUTH_URL` | `.env.local` (local) / App Service Settings (Azure) | Base URL of the application |
| `NEXTAUTH_SECRET` / `AUTH_SECRET` | `.env.local` (local) / Azure Key Vault (recommended for production) | JWT encryption secret |
| `AZURE_CLIENT_ID` | `.env.local` (local) / Azure Key Vault (production) | Application (client) ID from app registration |
| `AZURE_CLIENT_SECRET` | `.env.local` (local) / Azure Key Vault (production) | Client secret from app registration |
| `AZURE_TENANT_ID` | `.env.local` (local) / App Service Settings | Microsoft Entra ID tenant ID |
| `AZURE_SUBSCRIPTION_ID` | `.env.local` (local) / App Service Settings | Azure subscription for resource queries |

### Application Registration Setup

#### Step 1: Create App Registration

1. Navigate to [Azure Portal](https://portal.azure.com)
2. Go to **Microsoft Entra ID** > **App registrations** > **New registration**
3. Configure:
   - **Name**: `Pulse 360 Admin Portal`
   - **Supported account types**: Accounts in this organizational directory only (Single tenant)
   - **Redirect URI**: 
     - Type: Web
     - URI: `https://your-domain.com/api/auth/callback/azure-ad` (production)
     - URI: `http://localhost:3000/api/auth/callback/azure-ad` (development)

#### Step 2: Configure Authentication

1. In the app registration, go to **Authentication**
2. Add additional redirect URIs if needed
3. Under **Implicit grant and hybrid flows**, enable:
   - ✅ ID tokens (used for implicit and hybrid flows)
4. Under **Advanced settings**:
   - Allow public client flows: **No**

#### Step 3: Create Client Secret

1. Go to **Certificates & secrets** > **Client secrets** > **New client secret**
2. Description: `Pulse360-Production-Secret`
3. Expires: Choose appropriate expiration (recommended: 24 months)
4. **Copy the secret value immediately** and store it securely in Azure Key Vault or `.env.local`

#### Step 4: Configure API Permissions

Navigate to **API permissions** and add the following delegated permissions:

---

## Microsoft Graph API

**Base URL:** `https://graph.microsoft.com/v1.0`  
**Documentation:** [Microsoft Graph REST API Reference](https://learn.microsoft.com/en-us/graph/api/overview)

### Authentication

- **Method**: OAuth 2.0 On-Behalf-Of (OBO) flow
- **Token Scope**: `https://graph.microsoft.com/.default`
- **Token Acquisition**: Via `@azure/msal-node` using user's refresh token or access token
- **Token Lifetime**: Typically 60-90 minutes
- **Token Storage**: Not persisted; acquired on-demand per request

### Required API Permissions (Delegated)

Configure these in your app registration under **API permissions** > **Microsoft Graph** > **Delegated permissions**:

| Permission | Type | Reason | Admin Consent Required |
|------------|------|--------|----------------------|
| `User.Read` | Delegated | Read signed-in user's profile | No |
| `User.Read.All` | Delegated | Read all users' full profiles | Yes |
| `User.ReadWrite.All` | Delegated | Create and manage users | Yes |
| `Directory.Read.All` | Delegated | Read directory data (users, groups, apps) | Yes |
| `Directory.ReadWrite.All` | Delegated | Read and write directory data | Yes |
| `AuditLog.Read.All` | Delegated | Read audit logs | Yes |
| `Group.Read.All` | Delegated | Read all groups | Yes |
| `Group.ReadWrite.All` | Delegated | Create and manage groups | Yes |
| `Application.Read.All` | Delegated | Read applications and service principals | Yes |
| `Application.ReadWrite.All` | Delegated | Manage applications | Yes |
| `Sites.Read.All` | Delegated | Read SharePoint sites | Yes |
| `Sites.ReadWrite.All` | Delegated | Manage SharePoint sites | Yes |
| `Reports.Read.All` | Delegated | Read usage reports | Yes |
| `SecurityEvents.Read.All` | Delegated | Read security events | Yes |
| `SecurityActions.Read.All` | Delegated | Read security actions | Yes |
| `IdentityRiskEvent.Read.All` | Delegated | Read identity risk events | Yes |
| `IdentityRiskyUser.Read.All` | Delegated | Read risky users | Yes |
| `DeviceManagementManagedDevices.Read.All` | Delegated | Read Intune managed devices | Yes |
| `DeviceManagementConfiguration.Read.All` | Delegated | Read device configuration | Yes |
| `offline_access` | Delegated | Maintain access to data (refresh tokens) | No |
| `openid` | Delegated | Sign in and read user profile | No |
| `profile` | Delegated | View users' basic profile | No |
| `email` | Delegated | View users' email address | No |

### Common Endpoints Used

#### Users

- **List Users**: `GET /users`
  - **File**: `src/app/api/users/route.ts`
  - **Purpose**: Retrieve all users in the tenant
  - **Query Parameters**: `$select`, `$filter`, `$top`, `$skip`, `$orderby`, `$expand`
  - **Documentation**: [List users](https://learn.microsoft.com/en-us/graph/api/user-list)

- **Get User**: `GET /users/{id}`
  - **File**: `src/app/api/user-details/route.js`
  - **Purpose**: Get specific user details
  - **Documentation**: [Get user](https://learn.microsoft.com/en-us/graph/api/user-get)

- **Create User**: `POST /users`
  - **File**: `src/app/api/users/create-user/route.js`
  - **Purpose**: Create a new user
  - **Documentation**: [Create user](https://learn.microsoft.com/en-us/graph/api/user-post-users)

- **Get User Sign-in Activity**: `GET /users/{id}?$select=signInActivity`
  - **File**: Multiple user-related routes
  - **Documentation**: [signInActivity resource type](https://learn.microsoft.com/en-us/graph/api/resources/signinactivity)

#### Groups

- **List Groups**: `GET /groups`
  - **File**: `src/app/groups/page.jsx`
  - **Purpose**: Retrieve all groups
  - **Documentation**: [List groups](https://learn.microsoft.com/en-us/graph/api/group-list)

#### Applications

- **List Applications**: `GET /applications`
  - **File**: `src/app/applications/page.jsx`
  - **Purpose**: List app registrations
  - **Documentation**: [List applications](https://learn.microsoft.com/en-us/graph/api/application-list)

- **List Service Principals**: `GET /servicePrincipals`
  - **File**: `src/app/api/service-principals/route.js`
  - **Purpose**: List enterprise applications
  - **Documentation**: [List servicePrincipals](https://learn.microsoft.com/en-us/graph/api/serviceprincipal-list)

- **Add Password Credential**: `POST /applications/{id}/addPassword`
  - **File**: `src/app/api/applications/[id]/credentials/route.js`
  - **Purpose**: Create client secret
  - **Documentation**: [application: addPassword](https://learn.microsoft.com/en-us/graph/api/application-addpassword)

- **Remove Password Credential**: `POST /applications/{id}/removePassword`
  - **File**: `src/app/api/applications/[id]/remove-password/route.js`
  - **Purpose**: Delete client secret
  - **Documentation**: [application: removePassword](https://learn.microsoft.com/en-us/graph/api/application-removepassword)

#### Audit Logs

- **Directory Audit Logs**: `GET /auditLogs/directoryAudits`
  - **File**: `src/app/audit-logs/page.jsx`
  - **Purpose**: Retrieve audit log entries
  - **Query Parameters**: `$filter`, `$top`, `$orderby`
  - **Documentation**: [List directoryAudits](https://learn.microsoft.com/en-us/graph/api/directoryaudit-list)

- **Sign-in Logs**: `GET /auditLogs/signIns`
  - **File**: `src/app/api/user-signins/route.js`
  - **Purpose**: Retrieve user sign-in logs
  - **Documentation**: [List signIns](https://learn.microsoft.com/en-us/graph/api/signin-list)

- **Provisioning Logs**: `GET /auditLogs/provisioning`
  - **File**: `src/app/api/provisioning-logs/route.js`
  - **Purpose**: Retrieve provisioning logs
  - **Documentation**: [List provisioningObjectSummary](https://learn.microsoft.com/en-us/graph/api/provisioningobjectsummary-list)

#### Reports

- **Microsoft 365 Active User Counts**: `GET /reports/getOffice365ActiveUserCounts(period='D7')`
  - **File**: `src/app/api/reports/**/route.js`
  - **Documentation**: [getOffice365ActiveUserCounts](https://learn.microsoft.com/en-us/graph/api/reportroot-getoffice365activeuserdetail)

- **Teams User Activity**: `GET /reports/getTeamsUserActivityUserDetail(period='D7')`
  - **File**: `src/app/api/teams/user-activity/route.js`
  - **Documentation**: [getTeamsUserActivityUserDetail](https://learn.microsoft.com/en-us/graph/api/reportroot-getteamsuseractivityuserdetail)

#### Security

- **Secure Score**: `GET /security/secureScores`
  - **File**: `src/app/api/secure-score/route.js`
  - **Purpose**: Retrieve Microsoft Secure Score
  - **Documentation**: [List secureScores](https://learn.microsoft.com/en-us/graph/api/security-list-securescores)

- **Security Alerts**: `GET /security/alerts_v2`
  - **File**: Multiple security routes
  - **Documentation**: [List alerts](https://learn.microsoft.com/en-us/graph/api/security-list-alerts_v2)

- **Risky Users**: `GET /identityProtection/riskyUsers`
  - **File**: `src/app/api/risky-users/route.js`
  - **Documentation**: [List riskyUsers](https://learn.microsoft.com/en-us/graph/api/riskyuser-list)

#### Service Health

- **Service Health**: `GET /admin/serviceAnnouncement/healthOverviews`
  - **File**: `src/app/service-health/page.jsx`
  - **Purpose**: Get service health status
  - **Documentation**: [List serviceHealths](https://learn.microsoft.com/en-us/graph/api/serviceannouncement-list-healthoverviews)

- **Service Issues**: `GET /admin/serviceAnnouncement/issues`
  - **Documentation**: [List issues](https://learn.microsoft.com/en-us/graph/api/serviceannouncement-list-issues)

#### SharePoint

- **SharePoint Admin Settings**: `GET /admin/sharepoint/settings`
  - **File**: `src/app/api/sharepoint/sites/route.js`
  - **Purpose**: Get SharePoint tenant settings
  - **Documentation**: [Get settings](https://learn.microsoft.com/en-us/graph/api/sharepointsettings-get)

- **List Sites**: `GET /sites?search=*`
  - **Documentation**: [Search for sites](https://learn.microsoft.com/en-us/graph/api/site-search)

#### Intune (Device Management)

- **List Managed Devices**: `GET /deviceManagement/managedDevices`
  - **File**: `src/app/devices/page.jsx`
  - **Purpose**: Get all Intune-managed devices
  - **Documentation**: [List managedDevices](https://learn.microsoft.com/en-us/graph/api/intune-devices-manageddevice-list)

### Client Implementation

**File**: `src/lib/graphClient.js` and `src/app/lib/graphClient.ts`

```javascript
// Example: Acquiring Graph token for delegated access
async function getGraphToken() {
  const tokenRes = await fetch(
    `https://login.microsoftonline.com/${process.env.AZURE_TENANT_ID}/oauth2/v2.0/token`,
    {
      method: 'POST',
      headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
      body: new URLSearchParams({
        client_id: process.env.AZURE_CLIENT_ID,
        client_secret: process.env.AZURE_CLIENT_SECRET,
        scope: 'https://graph.microsoft.com/.default',
        grant_type: 'client_credentials',
      }),
    }
  );
  const data = await tokenRes.json();
  return data.access_token;
}
```

### Rate Limits

- **Throttling Threshold**: Varies by endpoint (typically 1,000-10,000 requests per minute per app)
- **Response Header**: `Retry-After` (in seconds)
- **Best Practice**: Implement exponential backoff on 429 (Too Many Requests) responses
- **Documentation**: [Microsoft Graph throttling guidance](https://learn.microsoft.com/en-us/graph/throttling)

---

## Azure Resource Manager API

**Base URL:** `https://management.azure.com`  
**Documentation:** [Azure REST API Reference](https://learn.microsoft.com/en-us/rest/api/azure/)

### Authentication

- **Method**: OAuth 2.0 Client Credentials flow
- **Token Scope**: `https://management.azure.com/.default`
- **Token Acquisition**: Via client credentials (app-only authentication)
- **Token Lifetime**: Typically 60-90 minutes

### Required API Permissions (Application)

Configure these in your app registration under **API permissions** > **Azure Service Management** > **Delegated permissions**:

| Permission | Type | Reason | Admin Consent Required |
|------------|------|--------|----------------------|
| `user_impersonation` | Delegated | Access Azure Resource Manager as the signed-in user | No |

Additionally, assign the application appropriate **Azure RBAC roles** at the subscription or resource group level:

| Azure Role | Scope | Reason |
|------------|-------|--------|
| **Reader** | Subscription | Read Azure resources and resource groups |
| **Contributor** | Subscription (optional) | Manage Azure resources |
| **Security Reader** | Subscription | Read security policies and states |

#### How to Assign Azure Roles

1. Navigate to [Azure Portal](https://portal.azure.com)
2. Go to **Subscriptions** > Select your subscription
3. Click **Access control (IAM)** > **Add role assignment**
4. Select role: **Reader**
5. Assign access to: **User, group, or service principal**
6. Search for your app registration name: `Pulse 360 Admin Portal`
7. Click **Save**

### Common Endpoints Used

#### Subscriptions

- **List Subscriptions**: `GET /subscriptions?api-version=2020-01-01`
  - **File**: `src/app/billing/page.jsx`
  - **Purpose**: List all accessible subscriptions
  - **Documentation**: [Subscriptions - List](https://learn.microsoft.com/en-us/rest/api/resources/subscriptions/list)

#### Azure Resource Graph

- **Query Resources**: `POST /providers/Microsoft.ResourceGraph/resources?api-version=2024-04-01`
  - **File**: `src/app/api/resource-graph/route.js`
  - **Purpose**: Query Azure resources using KQL (Kusto Query Language)
  - **Request Body**:
    ```json
    {
      "query": "Resources | where type == 'microsoft.compute/virtualmachines' | project name, location",
      "subscriptions": ["<subscription-id>"]
    }
    ```
  - **Documentation**: [Azure Resource Graph REST API](https://learn.microsoft.com/en-us/rest/api/azureresourcegraph/resourcegraph(2021-03-01)/resources/resources)

#### Azure Advisor

- **List Recommendations**: `GET /subscriptions/{subscriptionId}/providers/Microsoft.Advisor/recommendations?api-version=2023-01-01`
  - **File**: `src/app/api/azure-advisor-list/route.js`
  - **Purpose**: Get Azure Advisor recommendations
  - **Documentation**: [Recommendations - List](https://learn.microsoft.com/en-us/rest/api/advisor/recommendations/list)

- **Get Advisor Score**: `GET /subscriptions/{subscriptionId}/providers/Microsoft.Advisor/advisorScore?api-version=2023-01-01`
  - **File**: `src/app/api/azure-advisor-score/route.js`
  - **Documentation**: [Advisor Score](https://learn.microsoft.com/en-us/rest/api/advisor/advisor-score)

#### Cost Management

- **Query Costs**: `POST /subscriptions/{subscriptionId}/providers/Microsoft.CostManagement/query?api-version=2023-03-01`
  - **File**: `src/app/billing/page.jsx`
  - **Purpose**: Query cost and usage data
  - **Request Body**:
    ```json
    {
      "type": "ActualCost",
      "timeframe": "MonthToDate",
      "dataset": {
        "granularity": "Daily",
        "aggregation": {
          "totalCost": { "name": "Cost", "function": "Sum" }
        }
      }
    }
    ```
  - **Documentation**: [Query API](https://learn.microsoft.com/en-us/rest/api/cost-management/query/usage)

- **Usage Details**: `GET /subscriptions/{subscriptionId}/providers/Microsoft.Consumption/usageDetails?api-version=2024-08-01`
  - **Documentation**: [Usage Details - List](https://learn.microsoft.com/en-us/rest/api/consumption/usage-details/list)

#### Security (Microsoft Defender for Cloud)

- **List Security Assessments**: `GET /providers/Microsoft.Security/assessments?api-version=2024-09-01`
  - **File**: `src/app/api/security/assessments/route.js`
  - **Purpose**: Get security recommendations from Defender for Cloud
  - **Documentation**: [Assessments - List](https://learn.microsoft.com/en-us/rest/api/defenderforcloud/assessments/list)

- **Get Assessment**: `GET /subscriptions/{subscriptionId}/providers/Microsoft.Security/assessments/{assessmentName}?api-version=2024-09-01`
  - **File**: `src/app/api/security/assessments/get/route.js`
  - **Documentation**: [Assessments - Get](https://learn.microsoft.com/en-us/rest/api/defenderforcloud/assessments/get)

#### Log Analytics

- **List Workspaces**: `GET /subscriptions/{subscriptionId}/providers/Microsoft.OperationalInsights/workspaces?api-version=2023-09-01`
  - **File**: `src/app/api/log-analytics-workspaces/route.js`
  - **Purpose**: List Log Analytics workspaces
  - **Documentation**: [Workspaces - List](https://learn.microsoft.com/en-us/rest/api/loganalytics/workspaces/list)

- **Query Workspace**: `POST /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{workspaceName}/api/query?api-version=2023-09-01`
  - **File**: `src/app/api/log-analytics-query/route.js`
  - **Purpose**: Execute KQL query against workspace
  - **Request Body**:
    ```json
    {
      "query": "AzureActivity | take 10",
      "timespan": "P1D"
    }
    ```
  - **Documentation**: [Query - Execute](https://learn.microsoft.com/en-us/rest/api/loganalytics/dataaccess/query/execute)

#### Application Insights

- **Query Application Insights**: `POST /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Insights/components/{componentName}/api/query?api-version=2018-04-20`
  - **File**: `src/app/api/application-insights-query/route.js`
  - **Purpose**: Query Application Insights telemetry
  - **Documentation**: [Query API](https://learn.microsoft.com/en-us/rest/api/application-insights/query/execute)

- **Get Metrics**: `GET /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Insights/components/{componentName}/api/metrics/{metricId}?api-version=2018-04-20`
  - **File**: `src/app/api/application-insights-metrics/route.js`
  - **Documentation**: [Metrics - Get](https://learn.microsoft.com/en-us/rest/api/application-insights/metrics/get)

### Client Implementation

**File**: `src/lib/graphClient.js`

```javascript
async function getAzureToken() {
  const tokenRes = await fetch(
    `https://login.microsoftonline.com/${process.env.AZURE_TENANT_ID}/oauth2/v2.0/token`,
    {
      method: 'POST',
      headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
      body: new URLSearchParams({
        client_id: process.env.AZURE_CLIENT_ID,
        client_secret: process.env.AZURE_CLIENT_SECRET,
        scope: 'https://management.azure.com/.default',
        grant_type: 'client_credentials',
      }),
    }
  );
  const data = await tokenRes.json();
  return data.access_token;
}
```

### Rate Limits

- **General Limit**: 12,000 requests per hour per subscription
- **Read Operations**: 15,000 requests per hour
- **Write Operations**: 1,200 requests per hour
- **Documentation**: [Azure Resource Manager request limits](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/request-limits-and-throttling)

---

## Power Platform API

**Base URL:** `https://api.powerplatform.com`  
**Documentation:** [Power Platform API Reference](https://learn.microsoft.com/en-us/power-platform/admin/programmability-overview)

### Authentication

- **Method**: OAuth 2.0 On-Behalf-Of (OBO) flow
- **Token Scope**: `https://api.powerplatform.com/.default`
- **Token Acquisition**: Via MSAL OBO using user's access token or refresh token
- **Helper File**: `src/lib/ppToken.js`

### Required API Permissions (Delegated)

No explicit API permissions need to be added to the app registration for Power Platform API. Authentication relies on:

1. **User Permissions**: The signed-in user must have appropriate Power Platform admin roles
2. **Token Exchange**: The app exchanges the user's Microsoft Entra ID token for a Power Platform token

#### Required Power Platform Roles

Assign the following roles to users in the [Power Platform Admin Center](https://admin.powerplatform.microsoft.com):

| Role | Purpose |
|------|---------|
| **Power Platform Administrator** | Full access to all Power Platform resources |
| **Dynamics 365 Administrator** | Manage Dynamics 365 and Dataverse environments |
| **Environment Administrator** | Manage specific environments |

### Common Endpoints Used

#### Environments

- **List Environments**: `GET /environmentmanagement/environments?api-version=2024-10-01`
  - **File**: `src/app/power-platform/page.jsx`
  - **Purpose**: List all Power Platform environments
  - **Response Includes**:
    - Environment ID, name, display name
    - State (enabled/disabled)
    - Type (production, sandbox, trial, etc.)
    - Dataverse instance URL
    - Capacity information
    - Add-ons and features
  - **Documentation**: [List environments](https://learn.microsoft.com/en-us/rest/api/power-platform/environmentmanagement/environments/get-environments)

- **Get Environment**: `GET /environmentmanagement/environments/{environmentId}?api-version=2024-10-01`
  - **Purpose**: Get details for a specific environment
  - **Documentation**: [Get environment](https://learn.microsoft.com/en-us/rest/api/power-platform/environmentmanagement/environments/get-environment)

#### Tenant Settings

- **Get Tenant Settings**: `GET /tenantsettings?api-version=2024-10-01`
  - **Purpose**: Retrieve tenant-wide Power Platform settings
  - **Documentation**: [Tenant Settings API](https://learn.microsoft.com/en-us/power-platform/admin/programmability-tenant-settings)

### Client Implementation

**File**: `src/lib/ppToken.js`

```javascript
export async function getPowerPlatformDelegatedToken({ userAccessToken, userIdToken, refreshToken }) {
  const scopes = ['https://api.powerplatform.com/.default'];
  const app = getCca();
  
  // 1) Try OBO with access token
  if (userAccessToken) {
    try {
      const res = await app.acquireTokenOnBehalfOf({ oboAssertion: userAccessToken, scopes });
      if (res?.accessToken) return res.accessToken;
    } catch {}
  }
  
  // 2) Try OBO with id token
  if (userIdToken) {
    try {
      const res = await app.acquireTokenOnBehalfOf({ oboAssertion: userIdToken, scopes });
      if (res?.accessToken) return res.accessToken;
    } catch {}
  }
  
  // 3) Try refresh token exchange
  if (refreshToken) {
    try {
      return await acquireTokenByRefreshToken(refreshToken, scopes);
    } catch {}
  }
  
  throw new Error('Unable to get Power Platform delegated token');
}
```

### Rate Limits

- **Service Protection Limits**: Applies to Dataverse API (see Dataverse section)
- **General Guidance**: Implement retry logic for 429 and 503 responses
- **Documentation**: [Service protection API limits](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/api-limits)

### Additional Token Functions

**Power Automate Delegated Token** (NEW):

**File**: `src/lib/ppToken.js`

```javascript
export async function getPowerAutomateDelegatedToken({ userAccessToken, userIdToken, refreshToken }) {
  const scopes = ['https://service.flow.microsoft.com/.default'];
  const app = getCca();
  
  // Same OBO → refresh token fallback pattern as getPowerPlatformDelegatedToken
  // Returns token with audience: https://service.flow.microsoft.com
}
```

**Use Case**: Sending data to Power Automate flows with OAuth authentication ("Authenticated users" trigger option)

**BAP Delegated Token**:

```javascript
export async function getBAPDelegatedToken({ userAccessToken, userIdToken, refreshToken }) {
  const scopes = ['https://api.bap.microsoft.com/.default'];
  // Returns token for Business Application Platform APIs
}
```

**Use Case**: Alternative Power Platform management operations (less common, api.powerplatform.com preferred)

---

## Dataverse API

**Base URL:** `https://{environment-url}/api/data/v9.2/`  
**Example:** `https://org12345678.crm.dynamics.com/api/data/v9.2/`  
**Documentation:** [Dataverse Web API Reference](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/reference/about)

### Authentication

- **Method**: OAuth 2.0 On-Behalf-Of (OBO) flow
- **Token Scope**: `https://{environment-url}/.default` (dynamic based on environment)
- **Token Acquisition**: Via MSAL OBO using Power Platform token
- **Helper File**: `src/lib/ppToken.js`

### Important: Dynamic Token Scopes

Each Dataverse environment requires a **unique token** scoped to that environment's URL:

```javascript
// Example: Acquiring token for a specific environment
const instanceUrl = 'https://org12345678.crm.dynamics.com';
const scope = `${instanceUrl}/.default`;
const token = await acquireTokenOnBehalfOf([scope], userAssertion);
```

**⚠️ Critical**: Always normalize the instance URL by removing trailing slashes before constructing the scope:

```javascript
const normalizedUrl = instanceUrl.replace(/\/+$/, ''); // Remove trailing slashes
const scope = `${normalizedUrl}/.default`;
```

### Required Permissions

Dataverse permissions are managed through **Security Roles** within each environment:

1. **System Administrator**: Full access to all tables and operations
2. **System Customizer**: Create and manage custom tables
3. **Basic User**: Read and write own records
4. **Service Writer**: For application users (service-to-service authentication)

#### How to Assign Security Roles

1. Navigate to [Power Platform Admin Center](https://admin.powerplatform.microsoft.com)
2. Select **Environments** > Choose your environment
3. Go to **Settings** > **Users + permissions** > **Users**
4. Find your application user (must be created first)
5. Click **Manage security roles**
6. Assign **System Administrator** or appropriate role

#### Creating an Application User in Dataverse

For app-only authentication, create an application user:

1. In Power Platform Admin Center, go to your environment
2. Go to **Settings** > **Users + permissions** > **Application users**
3. Click **+ New app user**
4. Select your app registration (by Application ID)
5. Select business unit
6. Assign security roles
7. Click **Create**

### Common Endpoints Used

#### Teams

- **List Teams**: `GET /teams?$select=teamid,name,description,_businessunitid_value,modifiedon,_administratorid_value,teamtype,membershiptype,isdefault&$orderby=name asc`
  - **File**: `src/app/power-platform/environments/page.jsx` (line ~550)
  - **Purpose**: List all Dataverse teams in an environment
  - **Response Fields**:
    - `teamid`: Unique identifier (GUID)
    - `name`: Team name
    - `teamtype`: 0=Owner, 1=Access, 2=AAD Security Group, 3=AAD Office Group
    - `_administratorid_value`: Team administrator (systemuser GUID)
  - **Documentation**: [team entity reference](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/reference/entities/team)

- **Create Team**: `POST /teams`
  - **File**: `src/app/power-platform/environments/page.jsx` (line ~1786+)
  - **Request Body**:
    ```json
    {
      "name": "Sales Team",
      "description": "Team for sales operations",
      "businessunitid@odata.bind": "/businessunits(guid)",
      "administratorid@odata.bind": "/systemusers(guid)",
      "teamtype": 0
    }
    ```
  - **Documentation**: [Create team](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/create-entity-web-api)

- **Get Team Members**: `GET /teams({teamId})/teammembership_association?$select=systemuserid,fullname,internalemailaddress`
  - **File**: `src/app/power-platform/environments/page.jsx` (via API route)
  - **Purpose**: Get all members of a specific team
  - **Documentation**: [Retrieve related records](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/retrieve-related-entities-query)

- **Add Team Member**: `POST /teams({teamId})/teammembership_association/$ref`
  - **Request Body**:
    ```json
    {
      "@odata.id": "https://org.crm.dynamics.com/api/data/v9.2/systemusers(user-guid)"
    }
    ```
  - **Documentation**: [Associate entities](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/associate-disassociate-entities-using-web-api)

- **Remove Team Member**: `DELETE /teams({teamId})/teammembership_association({userId})/$ref`
  - **File**: `src/app/power-platform/environments/page.jsx` (line ~1154+)
  - **Documentation**: [Disassociate entities](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/associate-disassociate-entities-using-web-api#disassociate-entities)

#### Solutions

- **List Solutions**: `GET /solutions?$select=friendlyname,uniquename,version,ismanaged,createdon,modifiedon,installedon&$orderby=friendlyname asc`
  - **File**: `src/app/power-platform/environments/page.jsx` (via modal)
  - **Purpose**: List installed Dataverse solutions
  - **Documentation**: [solution entity reference](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/reference/entities/solution)

#### Bots (Copilot Studio Agents)

- **List Bots**: `GET /bots?$select=name,schemaname,publishedon,statecode,statuscode`
  - **File**: `src/app/power-platform/environments/page.jsx` (line ~1600+)
  - **Purpose**: List Copilot Studio agents (formerly Power Virtual Agents)
  - **Documentation**: [bot entity reference](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/reference/entities/bot)

#### Business Units

- **List Business Units**: `GET /businessunits?$select=businessunitid,name`
  - **File**: `src/app/power-platform/environments/page.jsx` (for team creation)
  - **Purpose**: List organizational units
  - **Documentation**: [businessunit entity reference](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/reference/entities/businessunit)

#### System Users

- **List Users**: `GET /systemusers?$select=systemuserid,fullname,internalemailaddress`
  - **File**: `src/app/power-platform/environments/page.jsx` (for admin selection)
  - **Purpose**: List Dataverse users
  - **Documentation**: [systemuser entity reference](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/reference/entities/systemuser)

#### Conversations (Transcripts)

- **List Conversations**: `GET /conversationtranscripts?$select=...&$orderby=createdon desc`
  - **File**: `src/app/power-platform/environments/page.jsx` (line ~2350+)
  - **Purpose**: Retrieve conversation transcripts from Copilot Studio
  - **Documentation**: [conversationtranscript entity reference](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/reference/entities/conversationtranscript)

#### Tables (EntityDefinitions)

- **List Tables**: `GET /EntityDefinitions?$select=LogicalName,SchemaName,EntitySetName,DisplayName,PrimaryNameAttribute`
  - **File**: `src/app/api/power-platform/dataverse/tables/route.js`
  - **Purpose**: List all table metadata
  - **Documentation**: [EntityMetadata](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/reference/entitymetadata)

- **Query Table Rows**: `GET /{entitySetName}?$select=field1,field2&$filter=...&$orderby=...&$top=50`
  - **File**: `src/app/api/power-platform/dataverse/query/route.js`
  - **Purpose**: Query records from any Dataverse table
  - **Query Parameters**:
    - `$select`: Comma-separated field names
    - `$filter`: OData filter expression
    - `$orderby`: Sort order (e.g., `createdon desc`)
    - `$top`: Limit number of results
    - `$expand`: Include related records
  - **Documentation**: [Query data using the Web API](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/query-data-web-api)

### OData Query Parameters

Dataverse uses **OData 4.0** query syntax:

| Parameter | Purpose | Example |
|-----------|---------|---------|
| `$select` | Choose specific fields | `$select=name,createdon` |
| `$filter` | Filter results | `$filter=statecode eq 0` |
| `$orderby` | Sort results | `$orderby=createdon desc` |
| `$top` | Limit results | `$top=50` |
| `$skip` | Skip results (pagination) | `$skip=50` |
| `$expand` | Include related records | `$expand=ownerid($select=fullname)` |
| `$count` | Include total count | `$count=true` |

**Documentation**: [Query data using OData](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/query-data-web-api)

### Pagination

Dataverse returns paginated results with a `@odata.nextLink` property:

```javascript
// First request
const response = await fetch(`${instanceUrl}/api/data/v9.2/accounts?$top=50`);
const data = await response.json();

// Check for next page
if (data['@odata.nextLink']) {
  const nextPageUrl = data['@odata.nextLink'];
  // Fetch next page...
}
```

### Rate Limits and Service Protection

Dataverse enforces **service protection limits**:

- **API requests per user per 5 minutes**: 6,000
- **Concurrent requests per user**: 52
- **Execution time per request**: 2 minutes

**429 Response**: When limits are exceeded, Dataverse returns HTTP 429 with a `Retry-After` header (in seconds).

**Best Practice**: Implement exponential backoff:

```javascript
async function fetchWithRetry(url, options, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    const response = await fetch(url, options);
    if (response.status === 429) {
      const retryAfter = parseInt(response.headers.get('Retry-After') || '60', 10);
      await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
      continue;
    }
    return response;
  }
  throw new Error('Max retries exceeded');
}
```

**Documentation**: [Service protection API limits](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/api-limits)

---

## Microsoft Entra ID (Azure AD)

**Token Endpoint:** `https://login.microsoftonline.com/{tenant-id}/oauth2/v2.0/token`  
**Documentation:** [Microsoft identity platform documentation](https://learn.microsoft.com/en-us/azure/active-directory/develop/)

### Authentication Flows

#### 1. Authorization Code Flow (User Sign-In)

Used by NextAuth.js for interactive user login:

1. User clicks "Sign In"
2. Browser redirects to: `https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?client_id=...&response_type=code&scope=openid+profile+email+offline_access&redirect_uri=...`
3. User authenticates and consents
4. Microsoft Entra ID redirects back with authorization code
5. Backend exchanges code for tokens

**File**: `src/lib/auth.ts` (NextAuth configuration)

#### 2. On-Behalf-Of (OBO) Flow

Used to exchange user token for resource-specific tokens:

```http
POST https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

client_id={client-id}
&client_secret={client-secret}
&grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer
&assertion={user-access-token}
&scope=https://graph.microsoft.com/.default
&requested_token_use=on_behalf_of
```

**File**: `src/lib/obo.js`

**Documentation**: [Microsoft identity platform and OAuth 2.0 On-Behalf-Of flow](https://learn.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow)

#### 3. Client Credentials Flow (App-Only)

Used for background services without user context:

```http
POST https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

client_id={client-id}
&client_secret={client-secret}
&scope=https://graph.microsoft.com/.default
&grant_type=client_credentials
```

**File**: `src/lib/graphClient.js`

**Documentation**: [Microsoft identity platform and the OAuth 2.0 client credentials flow](https://learn.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow)

#### 4. Refresh Token Flow

Used to acquire new access tokens without user interaction:

```http
POST https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

client_id={client-id}
&client_secret={client-secret}
&refresh_token={refresh-token}
&scope=https://api.powerplatform.com/.default
&grant_type=refresh_token
```

**File**: `src/lib/ppToken.js`

**Documentation**: [Refresh the access token](https://learn.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-auth-code-flow#refresh-the-access-token)

### Token Lifetimes

| Token Type | Default Lifetime | Configurable | Refresh Strategy |
|------------|------------------|--------------|------------------|
| Access Token | 60-90 minutes | Yes (via token lifetime policies) | Use refresh token |
| ID Token | 60-90 minutes | Yes | Obtain during new sign-in |
| Refresh Token | 90 days (rolling) | Yes | Automatically renewed on use |

**Documentation**: [Configurable token lifetimes](https://learn.microsoft.com/en-us/azure/active-directory/develop/active-directory-configurable-token-lifetimes)

---

## Application Insights API

**Base URL:** `https://api.applicationinsights.io/v1/apps/{app-id}`  
**Documentation:** [Application Insights REST API](https://dev.applicationinsights.io/reference)

### Authentication

- **Method**: Azure Resource Manager token
- **Token Scope**: `https://management.azure.com/.default`
- **Required Role**: Reader or Monitoring Reader on the Application Insights resource

### Common Endpoints Used

- **Query**: `GET /query?query={kql-query}`
  - **File**: `src/app/api/application-insights-query/route.js`
  - **Purpose**: Execute KQL query
  - **Example Query**: `exceptions | where timestamp > ago(24h) | summarize count() by type`
  - **Documentation**: [Query API](https://dev.applicationinsights.io/reference)

- **Metrics**: `GET /metrics/{metric-id}`
  - **File**: `src/app/api/application-insights-metrics/route.js`
  - **Purpose**: Get specific metric values
  - **Documentation**: [Metrics API](https://dev.applicationinsights.io/reference)

---

## Log Analytics API

**Base URL:** Via Azure Resource Manager  
**Documentation:** [Azure Monitor Logs REST API](https://learn.microsoft.com/en-us/rest/api/loganalytics/)

### Authentication

- **Method**: Azure Resource Manager token
- **Token Scope**: `https://management.azure.com/.default`
- **Required Role**: Reader or Log Analytics Reader on the workspace

### Common Endpoints Used

- **Query Workspace**: `POST /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{workspaceName}/api/query?api-version=2023-09-01`
  - **File**: `src/app/api/log-analytics-query/route.js`
  - **Request Body**:
    ```json
    {
      "query": "ContainerLog | where TimeGenerated > ago(1h)",
      "timespan": "PT1H"
    }
    ```
  - **Documentation**: [Query API](https://learn.microsoft.com/en-us/rest/api/loganalytics/dataaccess/query/execute)

---

## Internal API Routes

These are Next.js API routes that act as proxies to external APIs. They handle authentication, token acquisition, and error handling.

### Route Structure

All internal API routes are located in `src/app/api/` with the following pattern:

```
src/app/api/
├── graph/              # Microsoft Graph proxies
├── power-platform/     # Power Platform proxies
│   ├── dataverse/
│   │   ├── teams/
│   │   ├── tables/
│   │   ├── query/
│   │   └── solutions/
│   │       └── export/      # Solution export operations (NEW)
│   ├── environments/
│   │   └── {environmentId}/
│   │       ├── enable-managed/   # Enable managed environment (NEW)
│   │       ├── disable-managed/  # Disable managed environment (NEW)
│   │       └── delete/           # Delete environment (NEW)
│   ├── inventory/      # Tenant inventory queries (NEW)
│   └── send-to-flow/   # Power Automate integration (NEW)
├── resource-graph/     # Azure Resource Graph
├── security/           # Security & Compliance
└── ...
```

### Example Route Implementation

**File**: `src/app/api/power-platform/dataverse/teams/route.js`

```javascript
import { auth } from '@/lib/auth';
import { getPowerPlatformDelegatedToken } from '@/lib/ppToken';

export async function GET(request) {
  const session = await auth();
  if (!session) {
    return new Response('Unauthorized', { status: 401 });
  }

  const { searchParams } = new URL(request.url);
  const instanceUrl = searchParams.get('instanceUrl');
  
  // Get Power Platform token for this environment
  const token = await getPowerPlatformDelegatedToken({
    userAccessToken: session.accessToken,
    userIdToken: session.idToken,
    refreshToken: session.refreshToken,
  });

  // Call Dataverse API
  const response = await fetch(
    `${instanceUrl}/api/data/v9.2/teams?$select=teamid,name`,
    {
      headers: {
        'Authorization': `Bearer ${token}`,
        'Accept': 'application/json',
        'OData-MaxVersion': '4.0',
        'OData-Version': '4.0',
      },
    }
  );

  const data = await response.json();
  return Response.json(data);
}
```

### Common Route Patterns

#### 1. Authenticated Route with Session

```javascript
import { auth } from '@/lib/auth';

export async function GET(request) {
  const session = await auth();
  if (!session?.accessToken) {
    return new Response('Unauthorized', { status: 401 });
  }
  // ... use session.accessToken
}
```

#### 2. Client Credentials Route (App-Only)

```javascript
import { getAzureToken } from '@/lib/graphClient';

export async function GET(request) {
  const token = await getAzureToken(); // Client credentials flow
  // ... use token for Azure Resource Manager API
}
```

#### 3. OBO Route (User-Delegated)

```javascript
import { auth } from '@/lib/auth';
import { acquireOboToken } from '@/lib/obo';

export async function GET(request) {
  const session = await auth();
  const token = await acquireOboToken(
    ['https://graph.microsoft.com/.default'],
    session.accessToken
  );
  // ... use token for Microsoft Graph API
}
```

---

## Microsoft Defender for Endpoint API

### Overview

Microsoft Defender for Endpoint (formerly Microsoft Defender ATP) provides security-related data including alerts, machines, vulnerabilities, and recommendations.

### Base URL

```
https://api.securitycenter.microsoft.com
```

### Authentication

- **Method**: Client Credentials Flow
- **Token Endpoint**: `https://login.microsoftonline.com/{tenantId}/oauth2/v2.0/token`
- **Scope**: `https://api.securitycenter.microsoft.com/.default`

### Environment Variables

```bash
DEFENDER_ATP_CLIENT_ID=<your-defender-app-client-id>
DEFENDER_ATP_CLIENT_SECRET=<your-defender-app-client-secret>
```

### API Routes

**File**: `src/app/api/defender-atp/route.js`

**Endpoints Used**:
- **GET /api/alerts** - Retrieve security alerts
- **GET /api/machines** - List all machines
- **GET /api/vulnerabilities** - Get vulnerability data
- **GET /api/recommendations** - Security recommendations

### Required Permissions

Application Permissions:
- `SecurityAlert.Read.All` - Read all security alerts
- `Machine.Read.All` - Read all machine information
- `Vulnerability.Read.All` - Read vulnerability data
- `SecurityRecommendation.Read.All` - Read security recommendations

### Usage

```javascript
const response = await fetch('/api/defender-atp', {
  method: 'GET',
  headers: {
    'Content-Type': 'application/json'
  }
})
const data = await response.json()
// Returns: { alerts, machines, vulnerabilities, recommendations }
```

### Documentation

- [Defender for Endpoint API Reference](https://learn.microsoft.com/en-us/defender-endpoint/api/apis-intro)
- [Authentication Guide](https://learn.microsoft.com/en-us/defender-endpoint/api/exposed-apis-create-app-webapp)

---

## Microsoft Graph Reports API

### Overview

The reportRoot API provides access to 70+ Microsoft 365 usage and activity reports covering all services including OneDrive, SharePoint, Teams, Outlook, and more.

### Base URL

```
https://graph.microsoft.com/v1.0/reports
https://graph.microsoft.com/beta/reports (for beta features)
```

### Authentication

- **Method**: User-delegated or Application credentials
- **Token Scope**: `https://graph.microsoft.com/.default`
- **Required Permission**: `Reports.Read.All`

### Report Categories

1. **Microsoft 365 Active Users** - Daily active users by product
2. **Microsoft 365 Apps Usage** - Desktop and mobile app usage
3. **OneDrive Activity** - File activity and user engagement
4. **OneDrive Usage** - Storage consumption statistics
5. **SharePoint Activity** - Site and file interactions
6. **SharePoint Site Usage** - Site storage and page views
7. **Outlook Activity** - Email send/read/receive metrics
8. **Outlook App Usage** - Email client usage
9. **Mailbox Usage** - Mailbox storage and quota
10. **Teams User Activity** - Messages, calls, meetings
11. **Teams Device Usage** - Device types used for Teams
12. **Microsoft 365 Groups** - Group activity and storage
13. **Microsoft 365 Activations** - Office activation counts
14. **Copilot Usage** (Beta) - Copilot adoption metrics
15. **Forms Activity** (Beta) - Forms usage statistics
16. **Browser Usage** (Beta) - Microsoft Edge usage
17. **Viva Engage (Yammer)** - Yammer activity

### Generic Report Route Helper

**File**: `src/app/api/reports/createGraphReportRoute.js`

Provides a reusable helper function for creating report API routes:

```javascript
import { createGraphReportRoute } from '@/app/api/reports/createGraphReportRoute'

export const GET = createGraphReportRoute({
  endpoint: 'getOneDriveActivityUserDetail',
  supportsPeriod: false,
  supportsDate: true
})
```

### Report Parameters

**Period-based reports** support: `D7`, `D30`, `D90`, `D180`
```
GET /reports/getOffice365ActiveUserCounts(period='D30')
```

**Date-based reports** support: `YYYY-MM-DD`
```
GET /reports/getOneDriveActivityUserDetail(date=2025-11-29)
```

### Frontend Component

**File**: `src/app/components/GraphReportPage.jsx`

Reusable React component for rendering any Graph report with:
- Period/date selection
- CSV export
- Dark mode support
- Automatic data formatting

### Documentation

- [reportRoot API Reference](https://learn.microsoft.com/en-us/graph/api/resources/reportroot?view=graph-rest-beta)
- [Implementation Guide](docs/reports-implementation-guide.md)

---

## Windows Update State API (Intune)

### Overview

The Windows Update State API provides detailed information about Windows Update status for managed devices, including KB article numbers, update states, quality update version, and feature update version.

### Base URL

```
https://graph.microsoft.com/beta/deviceManagement/managedDevices/{deviceId}/windowsUpdateStates
```

### Authentication

- **Method**: User-delegated
- **Token Scope**: `https://graph.microsoft.com/.default`
- **Required Permission**: `DeviceManagementManagedDevices.Read.All`

### API Routes

**File**: `src/app/api/intune/windows-update-states/route.js`

**Frontend**: `src/app/intune/windows-updates/page.jsx`

### Query Parameters

- `deviceId` (required) - Intune managed device ID
- `$select` - Specific properties to retrieve
- `$filter` - Filter results
- `$expand` - Include related entities

### Example Usage

```javascript
const response = await fetch(
  '/api/intune/windows-update-states?deviceId=12345678-1234-1234-1234-123456789012'
)
const data = await response.json()
```

### Response Properties

| Property | Type | Description |
|----------|------|-------------|
| `id` | string | Update state record ID |
| `deviceId` | string | Device identifier |
| `deviceDisplayName` | string | Device name |
| `qualityUpdateVersion` | string | Current quality update version (e.g., "10.0.19045.5131") |
| `featureUpdateVersion` | string | Current feature update version (e.g., "22H2") |
| `lastScanDateTime` | datetime | Last Windows Update scan |
| `lastSyncDateTime` | datetime | Last sync with Intune |

### Documentation

- [windowsUpdateState resource type](https://learn.microsoft.com/en-us/graph/api/resources/intune-shared-windowsupdatestate?view=graph-rest-beta)
- [Get windowsUpdateState](https://learn.microsoft.com/en-us/graph/api/intune-shared-windowsupdatestate-get?view=graph-rest-beta)

---

## ServiceNow Integration API

### Overview

ServiceNow integration enables creating incidents directly from the portal based on security alerts, compliance issues, or other events detected in Microsoft 365/Azure.

### Authentication

ServiceNow uses **Basic Authentication** with username/password credentials.

### Environment Variables (Optional)

```bash
SERVICENOW_INSTANCE=<your-instance-name>
SERVICENOW_USERNAME=<service-account-username>
SERVICENOW_PASSWORD=<service-account-password>
```

Note: Credentials can also be provided dynamically via the UI.

### API Routes

#### 1. Fetch Incidents

**File**: `src/app/api/servicenow/incidents/route.js`

**Method**: POST (with credentials in body)

**Request Body**:
```json
{
  "instance": "dev12345",
  "username": "admin",
  "password": "password"
}
```

**Response**:
```json
{
  "result": [
    {
      "sys_id": "abc123",
      "number": "INC0001234",
      "short_description": "Issue description",
      "state": "2",
      "priority": "3",
      "assigned_to": "John Doe",
      "opened_at": "2025-11-29 10:30:00"
    }
  ]
}
```

#### 2. Create Incident

**File**: `src/app/api/servicenow/incidents/create/route.js`

**Method**: POST

**Request Body**:
```json
{
  "instance": "dev12345",
  "username": "admin",
  "password": "password",
  "incident": {
    "short_description": "Security alert detected",
    "description": "High severity alert...",
    "urgency": "1",
    "impact": "2",
    "category": "security",
    "assignment_group": "Security Team"
  }
}
```

**Response**:
```json
{
  "result": {
    "sys_id": "xyz789",
    "number": "INC0001235",
    "short_description": "Security alert detected",
    "state": "1"
  }
}
```

### Frontend Component

**File**: `src/app/components/CreateIncidentButton.jsx`

Reusable button component that can be placed anywhere to trigger incident creation:

```jsx
import CreateIncidentButton from '@/app/components/CreateIncidentButton'

<CreateIncidentButton
  title="Security Alert Detected"
  description={`Alert: ${alertData.name}\nSeverity: ${alertData.severity}`}
  urgency="1"
  impact="2"
  category="security"
/>
```

### ServiceNow API Endpoints

**Base URL**: `https://{instance}.service-now.com/api/now`

**Endpoints Used**:
- **GET /table/incident** - Fetch incidents
- **POST /table/incident** - Create incident
- **PATCH /table/incident/{sys_id}** - Update incident

### Documentation

- [ServiceNow REST API Reference](https://developer.servicenow.com/dev.do#!/reference/api/xanadu/rest/)
- [Incident Table API](https://developer.servicenow.com/dev.do#!/reference/api/xanadu/rest/c_TableAPI)
- [Implementation Guide](docs/servicenow-incident-integration.md)

---

## Required Permissions Summary

### App Registration Permissions

| API | Permission Name | Type | Admin Consent | Purpose |
|-----|----------------|------|---------------|---------|
| **Microsoft Graph** | User.Read.All | Delegated | Yes | Read all users |
| | User.ReadWrite.All | Delegated | Yes | Create/update users |
| | Directory.Read.All | Delegated | Yes | Read directory |
| | AuditLog.Read.All | Delegated | Yes | Read audit logs |
| | Group.Read.All | Delegated | Yes | Read groups |
| | Application.Read.All | Delegated | Yes | Read applications |
| | Sites.Read.All | Delegated | Yes | Read SharePoint |
| | Reports.Read.All | Delegated | Yes | Read usage reports |
| | SecurityEvents.Read.All | Delegated | Yes | Read security events |
| | DeviceManagementManagedDevices.Read.All | Delegated | Yes | Read Intune devices |
| | offline_access | Delegated | No | Get refresh tokens |
| **Azure Service Management** | user_impersonation | Delegated | No | Access ARM as user |
| **Microsoft Defender for Endpoint** | SecurityAlert.Read.All | Application | Yes | Read security alerts |
| | Machine.Read.All | Application | Yes | Read machine information |
| | Vulnerability.Read.All | Application | Yes | Read vulnerabilities |
| | SecurityRecommendation.Read.All | Application | Yes | Read security recommendations |

### Azure RBAC Roles

| Scope | Role | Purpose |
|-------|------|---------|
| Subscription | Reader | Read Azure resources |
| Subscription | Security Reader | Read security assessments |
| Log Analytics Workspace | Log Analytics Reader | Query logs |
| Application Insights | Monitoring Reader | Query metrics |

### Power Platform Roles

| Role | Assigned In | Purpose |
|------|-------------|---------|
| Power Platform Administrator | Microsoft 365 Admin Center | Manage all environments |
| Dynamics 365 Administrator | Microsoft 365 Admin Center | Manage Dataverse |
| System Administrator (Dataverse) | Power Platform Admin Center | Full access to Dataverse |

### Granting Admin Consent

After adding API permissions to your app registration:

1. Navigate to **API permissions** in your app registration
2. Click **Grant admin consent for {Tenant Name}**
3. Confirm by clicking **Yes**
4. Verify all permissions show **Granted for {Tenant Name}** in the Status column

**⚠️ Important**: Admin consent is required for most Microsoft Graph permissions. Without it, users will see consent prompts and may be unable to access features.

---

## Troubleshooting

### Common Error Messages

#### 1. "Unauthorized" (401)

**Cause**: Missing or invalid access token

**Solutions**:
- Verify `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, and `AZURE_TENANT_ID` are set correctly
- Check that the user is signed in (session exists)
- Verify token has not expired
- Check that refresh token is present in session

**Debug**:
```javascript
const session = await auth();
console.log('Session:', JSON.stringify(session, null, 2));
```

#### 2. "Forbidden" (403)

**Cause**: Insufficient permissions

**Solutions**:
- Verify API permissions are configured in app registration
- Ensure admin consent has been granted
- Check Azure RBAC role assignments for the service principal
- Verify Power Platform admin roles for the user

**Check Permissions**:
1. Azure Portal > App registrations > Your app > API permissions
2. Verify all permissions show "Granted" status

#### 3. "Invalid audience" Token Error

**Cause**: Token scope does not match the API resource

**Solutions**:
- For Graph API: Use `https://graph.microsoft.com/.default`
- For ARM API: Use `https://management.azure.com/.default`
- For Power Platform API: Use `https://api.powerplatform.com/.default`
- For Dataverse: Use `https://{environment-url}/.default` (without trailing slash)

**Verify Token**:
```javascript
// Decode JWT to inspect audience (aud) claim
const parts = token.split('.');
const payload = JSON.parse(atob(parts[1]));
console.log('Token audience:', payload.aud);
```

#### 4. "Service protection limit exceeded" (429)

**Cause**: Too many requests to Dataverse API

**Solutions**:
- Implement exponential backoff retry logic
- Respect `Retry-After` header value
- Reduce concurrent requests
- Batch operations where possible

**Example Retry Logic**:
```javascript
async function fetchWithRetry(url, options) {
  const response = await fetch(url, options);
  if (response.status === 429) {
    const retryAfter = parseInt(response.headers.get('Retry-After') || '60', 10);
    await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
    return fetchWithRetry(url, options); // Retry
  }
  return response;
}
```

#### 5. "AADSTS50001: Resource not found"

**Cause**: Incorrect resource URI in token request

**Solutions**:
- Verify the resource URL is correct and registered in your tenant
- For Dataverse, ensure the environment URL is correct
- Check for typos in scope/resource parameters

#### 6. "AADSTS65001: User or administrator has not consented"

**Cause**: Admin consent not granted or user cannot consent

**Solutions**:
- Grant admin consent in app registration (API permissions)
- Ensure user has permission to consent to applications
- Check tenant consent policies

#### 7. Power Platform API returns 404 for environments

**Cause**: User does not have Power Platform admin role

**Solutions**:
- Assign **Power Platform Administrator** role in Microsoft 365 Admin Center
- Alternatively, assign **Environment Administrator** for specific environments

### Debugging Tips

#### Enable Verbose Logging

**MSAL Logging** (`src/lib/ppToken.js`):
```javascript
const cca = new ConfidentialClientApplication({
  auth: { /* ... */ },
  system: { 
    loggerOptions: { 
      loggerCallback(level, message, containsPii) {
        console.log(`[MSAL][${level}]`, message);
      },
      piiLoggingEnabled: false,
      logLevel: 'Verbose'
    } 
  },
});
```

#### Inspect Token Claims

```javascript
function decodeJwt(token) {
  const parts = token.split('.');
  const payload = JSON.parse(Buffer.from(parts[1], 'base64').toString());
  console.log('Token claims:', JSON.stringify(payload, null, 2));
  console.log('Audience:', payload.aud);
  console.log('Scopes:', payload.scp || payload.roles);
  console.log('Expiration:', new Date(payload.exp * 1000).toISOString());
}
```

#### Test Token Acquisition

Use the provided test scripts:

```bash
# Test Power Platform token acquisition
node whoami-powerplatform.js

# Test MSAL certificate authentication
node test-msal-cert.js
```

#### Check API Response Details

```javascript
const response = await fetch(url, options);
const text = await response.text();
console.log('Status:', response.status);
console.log('Headers:', Object.fromEntries(response.headers.entries()));
console.log('Body:', text);
```

### Useful Links

- **Azure Portal**: [https://portal.azure.com](https://portal.azure.com)
- **Microsoft 365 Admin Center**: [https://admin.microsoft.com](https://admin.microsoft.com)
- **Power Platform Admin Center**: [https://admin.powerplatform.microsoft.com](https://admin.powerplatform.microsoft.com)
- **Microsoft Entra ID Admin Center**: [https://entra.microsoft.com](https://entra.microsoft.com)
- **Microsoft Graph Explorer**: [https://developer.microsoft.com/graph/graph-explorer](https://developer.microsoft.com/graph/graph-explorer)
- **JWT Decoder**: [https://jwt.ms](https://jwt.ms)

---

## Appendix: Quick Reference

### Environment Variables Checklist

- [ ] `NEXTAUTH_URL` set to your application URL
- [ ] `NEXTAUTH_SECRET` or `AUTH_SECRET` generated (32+ characters)
- [ ] `AZURE_CLIENT_ID` from app registration
- [ ] `AZURE_CLIENT_SECRET` from app registration (stored securely)
- [ ] `AZURE_TENANT_ID` from Microsoft Entra ID
- [ ] `AZURE_SUBSCRIPTION_ID` (optional, for Azure Resource Graph)

### App Registration Checklist

- [ ] App registration created in Microsoft Entra ID
- [ ] Redirect URIs configured for authentication
- [ ] Client secret created and stored securely
- [ ] API permissions added for Microsoft Graph
- [ ] API permissions added for Azure Service Management
- [ ] Admin consent granted for all permissions
- [ ] Service principal assigned Azure RBAC roles (Reader, Security Reader)

### Power Platform Checklist

- [ ] User assigned **Power Platform Administrator** role
- [ ] Application user created in each Dataverse environment
- [ ] Application user assigned **System Administrator** security role
- [ ] Environments accessible via Power Platform Admin Center

### Testing Checklist

- [ ] User can sign in successfully
- [ ] Microsoft Graph API returns data (e.g., /users)
- [ ] Azure Resource Graph returns resources
- [ ] Power Platform API returns environments
- [ ] Dataverse API returns teams/solutions
- [ ] Error messages are actionable

---

**Document Maintained By**: IT Operations Team  
**Last Review Date**: November 1, 2025  
**Next Review Date**: February 1, 2026

For questions or updates, please contact your Azure/Microsoft 365 administrator or submit a pull request to this documentation.
