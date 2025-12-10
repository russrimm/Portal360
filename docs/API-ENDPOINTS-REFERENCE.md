# API Endpoints Quick Reference

**Quick lookup guide for all external API endpoints used in Pulse 360°**  
**Last Updated:** November 17, 2025

This document provides a flat reference of every external API endpoint called by the application, organized by feature area. Use this for quick troubleshooting and endpoint verification.

---

## Table of Contents

- [Users Management](#users-management)
- [Groups Management](#groups-management)
- [Applications & Service Principals](#applications--service-principals)
- [Audit & Sign-in Logs](#audit--sign-in-logs)
- [Security & Compliance](#security--compliance)
- [Microsoft 365 Reports](#microsoft-365-reports)
- [Service Health](#service-health)
- [SharePoint](#sharepoint)
- [Device Management (Intune)](#device-management-intune)
- [Azure Resources](#azure-resources)
- [Azure Costs & Billing](#azure-costs--billing)
- [Azure Security (Defender)](#azure-security-defender)
- [Azure Monitoring](#azure-monitoring)
- [Power Platform Environments](#power-platform-environments)
- [Dataverse (Power Platform)](#dataverse-power-platform)
- [External Data Sources](#external-data-sources)

---

## Users Management

### List All Users
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/users`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `User.Read.All`
- **Source File**: `src/app/api/users/route.ts`
- **Query Parameters**:
  - `$select`: Field selection (e.g., `displayName,userPrincipalName,mail`)
  - `$filter`: OData filter (e.g., `accountEnabled eq true`)
  - `$top`: Limit results (max 999)
  - `$skip`: Skip results for pagination
  - `$orderby`: Sort order
  - `$count`: Include count
  - `$expand`: Include related data
- **Response**: JSON array of user objects
- **Pagination**: Use `@odata.nextLink` for next page
- **Rate Limit**: Part of Graph API throttling (varies by tenant)
- **Documentation**: [Microsoft Graph - List users](https://learn.microsoft.com/en-us/graph/api/user-list)

### Get User Details
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/users/{id}`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `User.Read.All`
- **Source File**: `src/app/api/user-details/route.js`
- **Path Parameters**: `{id}` - User ID or UPN
- **Response**: Single user object with full details
- **Documentation**: [Microsoft Graph - Get user](https://learn.microsoft.com/en-us/graph/api/user-get)

### Create User
- **Endpoint**: `POST https://graph.microsoft.com/v1.0/users`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `User.ReadWrite.All`
- **Source File**: `src/app/api/users/create-user/route.js`
- **Request Body** (minimum):
  ```json
  {
    "accountEnabled": true,
    "displayName": "John Doe",
    "mailNickname": "jdoe",
    "userPrincipalName": "jdoe@contoso.com",
    "passwordProfile": {
      "forceChangePasswordNextSignIn": true,
      "password": "TempPassword123!"
    }
  }
  ```
- **Response**: Created user object
- **Documentation**: [Microsoft Graph - Create user](https://learn.microsoft.com/en-us/graph/api/user-post-users)

### Get User Recent Activities
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/users/{id}/activities/recent`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `UserActivity.ReadWrite.CreatedByApp`
- **Source File**: `src/app/api/users/recent-activities/route.js`
- **Response**: User activity timeline
- **Documentation**: [Microsoft Graph - List recent activities](https://learn.microsoft.com/en-us/graph/api/projectrome-get-recent-activities)

---

## Groups Management

### List All Groups
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/groups`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `Group.Read.All`
- **Source File**: `src/app/groups/page.jsx`
- **Query Parameters**: Same as users ($select, $filter, $top, etc.)
- **Response**: JSON array of group objects
- **Pagination**: Use `@odata.nextLink`
- **Documentation**: [Microsoft Graph - List groups](https://learn.microsoft.com/en-us/graph/api/group-list)

### Get Group Details
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/groups/{id}`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `Group.Read.All`
- **Response**: Single group object
- **Documentation**: [Microsoft Graph - Get group](https://learn.microsoft.com/en-us/graph/api/group-get)

### List Group Members
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/groups/{id}/members`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `GroupMember.Read.All`
- **Response**: Array of directory objects (users, groups, service principals)
- **Documentation**: [Microsoft Graph - List members](https://learn.microsoft.com/en-us/graph/api/group-list-members)

---

## Applications & Service Principals

### List Applications (App Registrations)
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/applications`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `Application.Read.All`
- **Source File**: `src/app/applications/page.jsx`
- **Query Parameters**: `$select`, `$filter`, `$top`, `$orderby`
- **Response**: Array of application objects
- **Documentation**: [Microsoft Graph - List applications](https://learn.microsoft.com/en-us/graph/api/application-list)

### List Service Principals (Enterprise Applications)
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/servicePrincipals`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `Application.Read.All`
- **Source File**: `src/app/api/service-principals/route.js`
- **Query Parameters**: `$select`, `$filter`, `$top`, `$orderby`
- **Response**: Array of service principal objects
- **Documentation**: [Microsoft Graph - List servicePrincipals](https://learn.microsoft.com/en-us/graph/api/serviceprincipal-list)

### Add Application Password (Create Secret)
- **Endpoint**: `POST https://graph.microsoft.com/v1.0/applications/{id}/addPassword`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `Application.ReadWrite.All`
- **Source File**: `src/app/api/applications/[id]/credentials/route.js`
- **Request Body**:
  ```json
  {
    "passwordCredential": {
      "displayName": "MySecret",
      "endDateTime": "2025-12-31T23:59:59Z"
    }
  }
  ```
- **Response**: Password credential object with secret value (only returned once)
- **Documentation**: [Microsoft Graph - application: addPassword](https://learn.microsoft.com/en-us/graph/api/application-addpassword)

### Remove Application Password (Delete Secret)
- **Endpoint**: `POST https://graph.microsoft.com/v1.0/applications/{id}/removePassword`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `Application.ReadWrite.All`
- **Source File**: `src/app/api/applications/[id]/remove-password/route.js`
- **Request Body**:
  ```json
  {
    "keyId": "credential-key-id-guid"
  }
  ```
- **Response**: 204 No Content on success
- **Documentation**: [Microsoft Graph - application: removePassword](https://learn.microsoft.com/en-us/graph/api/application-removepassword)

---

## Audit & Sign-in Logs

### List Directory Audit Logs
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/auditLogs/directoryAudits`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `AuditLog.Read.All`
- **Source File**: `src/app/audit-logs/page.jsx`
- **Query Parameters**:
  - `$filter`: e.g., `activityDateTime ge 2025-01-01T00:00:00Z`
  - `$top`: Limit results (max 1000)
  - `$orderby`: e.g., `activityDateTime desc`
- **Response**: Array of directory audit objects
- **Documentation**: [Microsoft Graph - List directoryAudits](https://learn.microsoft.com/en-us/graph/api/directoryaudit-list)

### List Sign-in Logs
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/auditLogs/signIns`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `AuditLog.Read.All`, `Directory.Read.All`
- **Source File**: `src/app/api/user-signins/route.js`
- **Query Parameters**: Same filtering as directory audits
- **Response**: Array of sign-in objects
- **Note**: Premium P1 or P2 license required
- **Documentation**: [Microsoft Graph - List signIns](https://learn.microsoft.com/en-us/graph/api/signin-list)

### List Provisioning Logs
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/auditLogs/provisioning`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `AuditLog.Read.All`
- **Source File**: `src/app/api/provisioning-logs/route.js`
- **Query Parameters**: Standard OData filters
- **Response**: Array of provisioning event objects
- **Documentation**: [Microsoft Graph - List provisioningObjectSummary](https://learn.microsoft.com/en-us/graph/api/provisioningobjectsummary-list)

---

## Security & Compliance

### Get Secure Score
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/security/secureScores`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `SecurityEvents.Read.All`
- **Source File**: `src/app/api/secure-score/route.js`
- **Query Parameters**: `$top=1` for latest score
- **Response**: Array of secure score objects with current and max scores
- **Documentation**: [Microsoft Graph - List secureScores](https://learn.microsoft.com/en-us/graph/api/security-list-securescores)

### List Security Alerts
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/security/alerts_v2`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `SecurityEvents.Read.All`
- **Source File**: Multiple security routes
- **Query Parameters**: `$filter`, `$top`, `$skip`
- **Response**: Array of security alert objects
- **Documentation**: [Microsoft Graph - List alerts_v2](https://learn.microsoft.com/en-us/graph/api/security-list-alerts_v2)

### List Risky Users
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/identityProtection/riskyUsers`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `IdentityRiskyUser.Read.All`
- **Source File**: `src/app/api/risky-users/route.js`
- **Query Parameters**: `$filter`, `$top`, `$select`
- **Response**: Array of risky user objects with risk levels
- **Note**: Requires Microsoft Entra ID Premium P2
- **Documentation**: [Microsoft Graph - List riskyUsers](https://learn.microsoft.com/en-us/graph/api/riskyuser-list)

### List Conditional Access Policies
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `Policy.Read.All`
- **Response**: Array of conditional access policy objects
- **Documentation**: [Microsoft Graph - List conditionalAccessPolicies](https://learn.microsoft.com/en-us/graph/api/conditionalaccessroot-list-policies)

---

## Microsoft 365 Reports

### Get Office 365 Active User Counts
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/reports/getOffice365ActiveUserCounts(period='D7')`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `Reports.Read.All`
- **Source File**: `src/app/api/reports/**/route.js`
- **Query Parameters**: `period` - D7, D30, D90, D180
- **Response**: CSV data with user counts by product
- **Documentation**: [Microsoft Graph - getOffice365ActiveUserCounts](https://learn.microsoft.com/en-us/graph/api/reportroot-getoffice365activeuserdetail)

### Get Teams User Activity
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/reports/getTeamsUserActivityUserDetail(period='D7')`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `Reports.Read.All`
- **Source File**: `src/app/api/teams/user-activity/route.js`
- **Query Parameters**: `period` - D7, D30, D90, D180
- **Response**: CSV data with Teams activity by user
- **Documentation**: [Microsoft Graph - getTeamsUserActivityUserDetail](https://learn.microsoft.com/en-us/graph/api/reportroot-getteamsuseractivityuserdetail)

### Get SharePoint Site Usage
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/reports/getSharePointSiteUsageDetail(period='D7')`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `Reports.Read.All`
- **Response**: CSV data with SharePoint site metrics
- **Documentation**: [Microsoft Graph Reports](https://learn.microsoft.com/en-us/graph/api/resources/report)

---

## Service Health

### List Service Health Overviews
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/admin/serviceAnnouncement/healthOverviews`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `ServiceHealth.Read.All`
- **Source File**: `src/app/service-health/page.jsx`
- **Response**: Array of service health status objects
- **Documentation**: [Microsoft Graph - List serviceHealths](https://learn.microsoft.com/en-us/graph/api/serviceannouncement-list-healthoverviews)

### List Service Issues
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/admin/serviceAnnouncement/issues`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `ServiceHealth.Read.All`
- **Query Parameters**: `$filter`, `$select`, `$top`
- **Response**: Array of active service issue objects
- **Documentation**: [Microsoft Graph - List issues](https://learn.microsoft.com/en-us/graph/api/serviceannouncement-list-issues)

### List Service Messages
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/admin/serviceAnnouncement/messages`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `ServiceMessage.Read.All`
- **Response**: Array of message center announcements
- **Documentation**: [Microsoft Graph - List messages](https://learn.microsoft.com/en-us/graph/api/serviceannouncement-list-messages)

---

## SharePoint

### Get SharePoint Admin Settings
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/admin/sharepoint/settings`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `SharePointTenantSettings.Read.All`
- **Source File**: `src/app/api/sharepoint/sites/route.js`
- **Response**: SharePoint tenant settings object
- **Documentation**: [Microsoft Graph - Get sharepointSettings](https://learn.microsoft.com/en-us/graph/api/sharepointsettings-get)

### Search SharePoint Sites
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/sites?search=*`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `Sites.Read.All`
- **Query Parameters**: `search` - Search query string
- **Response**: Array of site objects
- **Documentation**: [Microsoft Graph - Search for sites](https://learn.microsoft.com/en-us/graph/api/site-search)

---

## Device Management (Intune)

### List Managed Devices
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/deviceManagement/managedDevices`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `DeviceManagementManagedDevices.Read.All`
- **Source File**: `src/app/devices/page.jsx`
- **Query Parameters**: `$select`, `$filter`, `$top`, `$skip`
- **Response**: Array of managed device objects
- **Pagination**: Use `@odata.nextLink`
- **Documentation**: [Microsoft Graph - List managedDevices](https://learn.microsoft.com/en-us/graph/api/intune-devices-manageddevice-list)

### Get Device Details
- **Endpoint**: `GET https://graph.microsoft.com/v1.0/deviceManagement/managedDevices/{id}`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `DeviceManagementManagedDevices.Read.All`
- **Source File**: `src/app/api/device-management/route.js`
- **Response**: Single managed device object
- **Documentation**: [Microsoft Graph - Get managedDevice](https://learn.microsoft.com/en-us/graph/api/intune-devices-manageddevice-get)

### Remote Lock Device
- **Endpoint**: `POST https://graph.microsoft.com/v1.0/deviceManagement/managedDevices/{id}/remoteLock`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `DeviceManagementManagedDevices.PrivilegedOperations.All`
- **Request Body**: Empty
- **Response**: 204 No Content on success
- **Documentation**: [Microsoft Graph - remoteLock action](https://learn.microsoft.com/en-us/graph/api/intune-devices-manageddevice-remotelock)

### Wipe Device
- **Endpoint**: `POST https://graph.microsoft.com/v1.0/deviceManagement/managedDevices/{id}/wipe`
- **Authentication**: Microsoft Graph delegated token
- **Required Permission**: `DeviceManagementManagedDevices.PrivilegedOperations.All`
- **Request Body**: 
  ```json
  {
    "keepEnrollmentData": false,
    "keepUserData": false
  }
  ```
- **Response**: 204 No Content on success
- **Documentation**: [Microsoft Graph - wipe action](https://learn.microsoft.com/en-us/graph/api/intune-devices-manageddevice-wipe)

---

## Azure Resources

### Query Resources with Resource Graph
- **Endpoint**: `POST https://management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2024-04-01`
- **Authentication**: Azure Resource Manager token (scope: `https://management.azure.com/.default`)
- **Required Permission**: Azure Reader role on subscription
- **Source File**: `src/app/api/resource-graph/route.js`
- **Request Body**:
  ```json
  {
    "query": "Resources | where type == 'microsoft.compute/virtualmachines' | project name, location, resourceGroup",
    "subscriptions": ["subscription-id-here"]
  }
  ```
- **Response**: JSON with data array and query statistics
- **Documentation**: [Azure Resource Graph REST API](https://learn.microsoft.com/en-us/rest/api/azureresourcegraph/resourcegraph(2021-03-01)/resources/resources)

### List Subscriptions
- **Endpoint**: `GET https://management.azure.com/subscriptions?api-version=2020-01-01`
- **Authentication**: Azure Resource Manager token
- **Required Permission**: Reader role or higher
- **Source File**: `src/app/billing/page.jsx`
- **Response**: Array of subscription objects
- **Documentation**: [Azure Subscriptions - List](https://learn.microsoft.com/en-us/rest/api/resources/subscriptions/list)

---

## Azure Costs & Billing

### Query Cost Data
- **Endpoint**: `POST https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.CostManagement/query?api-version=2023-03-01`
- **Authentication**: Azure Resource Manager token
- **Required Permission**: Cost Management Reader or higher
- **Source File**: `src/app/billing/page.jsx`
- **Request Body**:
  ```json
  {
    "type": "ActualCost",
    "timeframe": "MonthToDate",
    "dataset": {
      "granularity": "Daily",
      "aggregation": {
        "totalCost": { "name": "Cost", "function": "Sum" }
      },
      "grouping": [
        { "type": "Dimension", "name": "ResourceGroup" }
      ]
    }
  }
  ```
- **Response**: Cost data with columns and rows
- **Documentation**: [Azure Cost Management - Query API](https://learn.microsoft.com/en-us/rest/api/cost-management/query/usage)

### Get Usage Details
- **Endpoint**: `GET https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Consumption/usageDetails?api-version=2024-08-01`
- **Authentication**: Azure Resource Manager token
- **Required Permission**: Cost Management Reader
- **Source File**: `src/app/billing/page.jsx`
- **Query Parameters**:
  - `$filter`: e.g., `properties/usageStart ge '2025-01-01'`
  - `$top`: Limit results
- **Response**: Array of usage detail objects
- **Documentation**: [Azure Consumption - Usage Details](https://learn.microsoft.com/en-us/rest/api/consumption/usage-details/list)

---

## Azure Security (Defender)

### List Security Assessments
- **Endpoint**: `GET https://management.azure.com/providers/Microsoft.Security/assessments?api-version=2024-09-01`
- **Authentication**: Azure Resource Manager token
- **Required Permission**: Security Reader role
- **Source File**: `src/app/api/security/assessments/route.js`
- **Query Parameters**: `$expand=links,metadata`
- **Response**: Array of security assessment objects
- **Documentation**: [Microsoft Defender - List assessments](https://learn.microsoft.com/en-us/rest/api/defenderforcloud/assessments/list)

### Get Security Assessment
- **Endpoint**: `GET https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Security/assessments/{assessmentName}?api-version=2024-09-01`
- **Authentication**: Azure Resource Manager token
- **Required Permission**: Security Reader role
- **Source File**: `src/app/api/security/assessments/get/route.js`
- **Path Parameters**: `{assessmentName}` - Assessment ID
- **Response**: Single assessment object with findings
- **Documentation**: [Microsoft Defender - Get assessment](https://learn.microsoft.com/en-us/rest/api/defenderforcloud/assessments/get)

### List Azure Advisor Recommendations
- **Endpoint**: `GET https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Advisor/recommendations?api-version=2023-01-01`
- **Authentication**: Azure Resource Manager token
- **Required Permission**: Reader role
- **Source File**: `src/app/api/azure-advisor-list/route.js`
- **Query Parameters**: `$filter`, `$top`, `$skip`
- **Response**: Array of recommendation objects
- **Documentation**: [Azure Advisor - List recommendations](https://learn.microsoft.com/en-us/rest/api/advisor/recommendations/list)

### Get Advisor Score
- **Endpoint**: `GET https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Advisor/advisorScore?api-version=2023-01-01`
- **Authentication**: Azure Resource Manager token
- **Required Permission**: Reader role
- **Source File**: `src/app/api/azure-advisor-score/route.js`
- **Response**: Advisor score object with category scores
- **Documentation**: [Azure Advisor Score](https://learn.microsoft.com/en-us/rest/api/advisor/advisor-score)

---

## Azure Monitoring

### List Log Analytics Workspaces
- **Endpoint**: `GET https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.OperationalInsights/workspaces?api-version=2023-09-01`
- **Authentication**: Azure Resource Manager token
- **Required Permission**: Reader role
- **Source File**: `src/app/api/log-analytics-workspaces/route.js`
- **Response**: Array of workspace objects
- **Documentation**: [Log Analytics - List workspaces](https://learn.microsoft.com/en-us/rest/api/loganalytics/workspaces/list)

### Query Log Analytics Workspace
- **Endpoint**: `POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{workspaceName}/api/query?api-version=2023-09-01`
- **Authentication**: Azure Resource Manager token
- **Required Permission**: Log Analytics Reader role
- **Source File**: `src/app/api/log-analytics-query/route.js`
- **Request Body**:
  ```json
  {
    "query": "AzureActivity | where TimeGenerated > ago(1h) | take 100",
    "timespan": "PT1H"
  }
  ```
- **Response**: Query results with tables and rows
- **Documentation**: [Log Analytics - Query](https://learn.microsoft.com/en-us/rest/api/loganalytics/dataaccess/query/execute)

### List Application Insights Instances
- **Endpoint**: `GET https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Insights/components?api-version=2020-02-02`
- **Authentication**: Azure Resource Manager token
- **Required Permission**: Reader role
- **Source File**: `src/app/api/application-insights-instances/route.js`
- **Response**: Array of Application Insights component objects
- **Documentation**: [Application Insights - List](https://learn.microsoft.com/en-us/rest/api/application-insights/components/list)

### Query Application Insights
- **Endpoint**: `POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Insights/components/{componentName}/api/query?api-version=2018-04-20`
- **Authentication**: Azure Resource Manager token
- **Required Permission**: Monitoring Reader role
- **Source File**: `src/app/api/application-insights-query/route.js`
- **Request Body**:
  ```json
  {
    "query": "exceptions | where timestamp > ago(24h) | summarize count() by type",
    "timespan": "P1D"
  }
  ```
- **Response**: Query results with tables
- **Documentation**: [Application Insights Query API](https://learn.microsoft.com/en-us/rest/api/application-insights/query/execute)

### Get Application Insights Metrics
- **Endpoint**: `GET https://api.applicationinsights.io/v1/apps/{app-id}/metrics/{metric-id}`
- **Authentication**: Azure Resource Manager token OR API key
- **Required Permission**: Monitoring Reader role
- **Source File**: `src/app/api/application-insights-metrics/route.js`
- **Path Parameters**: 
  - `{app-id}`: Application Insights application ID
  - `{metric-id}`: Metric name (e.g., `requests/count`)
- **Query Parameters**: `timespan`, `aggregation`
- **Response**: Metric values
- **Documentation**: [Application Insights Metrics API](https://dev.applicationinsights.io/reference)

---

## Power Platform Environments

### List Power Platform Environments
- **Endpoint**: `GET https://api.powerplatform.com/environmentmanagement/environments?api-version=2024-10-01`
- **Authentication**: Power Platform delegated token (scope: `https://api.powerplatform.com/.default`)
- **Required Permission**: Power Platform Administrator role
- **Source File**: `src/app/power-platform/page.jsx`
- **Response**: Array of environment objects with:
  - `id`, `name`, `properties.displayName`
  - `properties.linkedEnvironmentMetadata.instanceUrl` (Dataverse URL)
  - `properties.capacity` (storage, database capacity)
  - `properties.addons` (capacity add-ons)
  - `properties.environmentType` (Production, Sandbox, Trial, etc.)
- **Documentation**: [Power Platform API - List environments](https://learn.microsoft.com/en-us/rest/api/power-platform/environmentmanagement/environments/get-environments)

### Get Power Platform Tenant Settings
- **Endpoint**: `GET https://api.powerplatform.com/tenantsettings?api-version=2024-10-01`
- **Authentication**: Power Platform delegated token
- **Required Permission**: Power Platform Administrator role
- **Response**: Tenant settings object
- **Documentation**: [Power Platform Tenant Settings API](https://learn.microsoft.com/en-us/power-platform/admin/programmability-tenant-settings)

---

## Dataverse (Power Platform)

**Important**: All Dataverse endpoints use dynamic base URLs based on the environment:  
`https://{environment-url}/api/data/v9.2/`

Example: `https://org12345678.crm.dynamics.com/api/data/v9.2/`

### List Teams
- **Endpoint**: `GET {instanceUrl}/api/data/v9.2/teams`
- **Authentication**: Dataverse token scoped to `{instanceUrl}/.default`
- **Required Permission**: System Administrator security role in environment
- **Source File**: `src/app/power-platform/environments/page.jsx` (line ~550)
- **Query Parameters**:
  - `$select=teamid,name,description,_businessunitid_value,teamtype,_administratorid_value`
  - `$orderby=name asc`
  - `$filter`: OData filter expression
  - `$top`: Limit results
- **Response**: JSON object with `value` array of team objects
- **Documentation**: [Dataverse - team entity](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/reference/entities/team)

### Create Team
- **Endpoint**: `POST {instanceUrl}/api/data/v9.2/teams`
- **Authentication**: Dataverse token
- **Required Permission**: System Administrator security role
- **Source File**: `src/app/power-platform/environments/page.jsx` (line ~1786+)
- **Request Headers**:
  - `Content-Type: application/json`
  - `OData-MaxVersion: 4.0`
  - `OData-Version: 4.0`
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
- **Response**: 204 No Content with `OData-EntityId` header containing new team URL
- **Documentation**: [Dataverse - Create entity](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/create-entity-web-api)

### Get Team Members
- **Endpoint**: `GET {instanceUrl}/api/data/v9.2/teams({teamId})/teammembership_association`
- **Authentication**: Dataverse token
- **Required Permission**: Read access to team
- **Source File**: `src/app/api/power-platform/dataverse/teams/{teamId}/members/route.js`
- **Query Parameters**: `$select=systemuserid,fullname,internalemailaddress`
- **Response**: Array of systemuser objects
- **Documentation**: [Dataverse - Retrieve related entities](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/retrieve-related-entities-query)

### Add Team Member
- **Endpoint**: `POST {instanceUrl}/api/data/v9.2/teams({teamId})/teammembership_association/$ref`
- **Authentication**: Dataverse token
- **Required Permission**: Write access to team
- **Request Body**:
  ```json
  {
    "@odata.id": "{instanceUrl}/api/data/v9.2/systemusers({userId})"
  }
  ```
- **Response**: 204 No Content on success
- **Documentation**: [Dataverse - Associate entities](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/associate-disassociate-entities-using-web-api)

### Remove Team Member
- **Endpoint**: `DELETE {instanceUrl}/api/data/v9.2/teams({teamId})/teammembership_association({userId})/$ref`
- **Authentication**: Dataverse token
- **Required Permission**: Write access to team
- **Source File**: `src/app/power-platform/environments/page.jsx` (line ~1154+)
- **Response**: 204 No Content on success
- **Documentation**: [Dataverse - Disassociate entities](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/associate-disassociate-entities-using-web-api#disassociate-entities)

### List Solutions
- **Endpoint**: `GET {instanceUrl}/api/data/v9.2/solutions`
- **Authentication**: Dataverse token
- **Required Permission**: Read access to solutions
- **Query Parameters**: `$select=friendlyname,uniquename,version,ismanaged,createdon,modifiedon,installedon&$orderby=friendlyname asc`
- **Response**: Array of solution objects
- **Documentation**: [Dataverse - solution entity](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/reference/entities/solution)

### List Bots (Copilot Agents)
- **Endpoint**: `GET {instanceUrl}/api/data/v9.2/bots`
- **Authentication**: Dataverse token
- **Required Permission**: Read access to bot entity
- **Source File**: `src/app/power-platform/environments/page.jsx` (line ~1600+)
- **Query Parameters**: `$select=name,schemaname,publishedon,statecode,statuscode`
- **Response**: Array of bot objects
- **Note**: Application user must have read access to bot entity; assign appropriate security role
- **Documentation**: [Dataverse - bot entity](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/reference/entities/bot)

### List Business Units
- **Endpoint**: `GET {instanceUrl}/api/data/v9.2/businessunits`
- **Authentication**: Dataverse token
- **Required Permission**: Read access to business units
- **Query Parameters**: `$select=businessunitid,name&$filter=contains(name,'search-term')`
- **Response**: Array of business unit objects
- **Documentation**: [Dataverse - businessunit entity](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/reference/entities/businessunit)

### List System Users
- **Endpoint**: `GET {instanceUrl}/api/data/v9.2/systemusers`
- **Authentication**: Dataverse token
- **Required Permission**: Read access to users
- **Query Parameters**: `$select=systemuserid,fullname,internalemailaddress&$filter=contains(fullname,'search-term')`
- **Response**: Array of systemuser objects
- **Documentation**: [Dataverse - systemuser entity](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/reference/entities/systemuser)

### List Audit Logs
- **Endpoint**: `GET {instanceUrl}/api/data/v9.2/audits`
- **Authentication**: Dataverse token
- **Required Permission**: Read access to audit entity
- **Source File**: `src/app/power-platform/environments/page.jsx`
- **Query Parameters**: 
  - `$select=auditid,createdon,action,operation,objecttypecode,_objectid_value,_userid_value,attributemask`
  - `$orderby=createdon desc`
  - `$top=100`
- **Response**: Array of audit record objects with change tracking information
- **Note**: `_objectid_value` is the GUID of the audited record; use `objecttypecode` for entity type
- **Documentation**: [Dataverse - audit entity](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/reference/entities/audit)

### List Conversations (Transcripts)
- **Endpoint**: `GET {instanceUrl}/api/data/v9.2/conversationtranscripts`
- **Authentication**: Dataverse token
- **Required Permission**: Read access to conversation transcripts
- **Source File**: `src/app/power-platform/environments/page.jsx` (line ~2350+)
- **Query Parameters**: `$select=...&$orderby=createdon desc&$top=100`
- **Response**: Array of conversation transcript objects
- **Documentation**: [Dataverse - conversationtranscript entity](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/reference/entities/conversationtranscript)

### List Tables (EntityDefinitions)
- **Endpoint**: `GET {instanceUrl}/api/data/v9.2/EntityDefinitions`
- **Authentication**: Dataverse token
- **Required Permission**: Read metadata
- **Source File**: `src/app/api/power-platform/dataverse/tables/route.js`
- **Query Parameters**: `$select=LogicalName,SchemaName,EntitySetName,DisplayName,PrimaryNameAttribute`
- **Response**: Array of entity metadata objects
- **Documentation**: [Dataverse - EntityMetadata](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/reference/entitymetadata)

### Query Table Rows
- **Endpoint**: `GET {instanceUrl}/api/data/v9.2/{entitySetName}`
- **Authentication**: Dataverse token
- **Required Permission**: Read access to specific table
- **Source File**: `src/app/api/power-platform/dataverse/query/route.js`
- **Query Parameters**:
  - `$select=field1,field2,field3` - Select specific columns
  - `$filter=statecode eq 0` - Filter results
  - `$orderby=createdon desc` - Sort order
  - `$top=50` - Limit results
  - `$expand=ownerid($select=fullname)` - Include related data
  - `$count=true` - Include total count
- **Response**: JSON object with `value` array and optional `@odata.nextLink` for pagination
- **Documentation**: [Dataverse - Query data](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/query-data-web-api)

### Export Solution
- **Endpoint**: `POST {instanceUrl}/api/data/v9.2/ExportSolution`
- **Authentication**: Dataverse token
- **Required Permission**: System Administrator security role
- **Source File**: `src/app/api/power-platform/dataverse/solutions/export/route.js`
- **Request Body**:
  ```json
  {
    "SolutionName": "MyCustomSolution",
    "Managed": false,
    "ExportAutoNumberingSettings": true,
    "ExportCalendarSettings": true,
    "ExportCustomizationSettings": true,
    "ExportEmailTrackingSettings": true,
    "ExportGeneralSettings": true,
    "ExportMarketingSettings": true,
    "ExportOutlookSynchronizationSettings": true,
    "ExportRelationshipRoles": true,
    "ExportIsvConfig": true,
    "ExportSales": true,
    "ExportExternalApplications": true
  }
  ```
- **Response**: JSON with `ExportJobId` (GUID) and `AsyncOperationId` for status tracking
- **Documentation**: [Dataverse - ExportSolution Action](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/reference/exportsolution)

### Check Solution Export Status
- **Endpoint**: `GET {instanceUrl}/api/data/v9.2/asyncoperations({asyncOperationId})`
- **Authentication**: Dataverse token
- **Required Permission**: System Administrator security role
- **Source File**: `src/app/api/power-platform/dataverse/solutions/export/status/route.js`
- **Query Parameters**: `$select=statuscode,statecode,message,percentcomplete`
- **Response**: Async operation status object
- **Retry Logic**: 20-second timeout with 2 retries on connection failure
- **Documentation**: [Dataverse - asyncoperation entity](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/reference/entities/asyncoperation)

### Download Exported Solution
- **Endpoint**: `GET {instanceUrl}/api/data/v9.2/exportsolutionuploads({exportJobId})`
- **Authentication**: Dataverse token
- **Required Permission**: System Administrator security role
- **Source File**: `src/app/api/power-platform/dataverse/solutions/export/download/route.js`
- **Query Parameters**: `$select=exportsolutionuploadid,name,createdon,solutionfile`
- **Response**: JSON object with base64-encoded solution file in `solutionfile` field
- **Note**: Solution file is a .zip archive encoded in base64
- **Documentation**: [Dataverse - exportsolutionupload entity](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/reference/entities/exportsolutionupload)

### Enable Managed Environment
- **Endpoint**: `POST https://api.powerplatform.com/environmentmanagement/environments/{environmentId}/enableManagedEnvironment?api-version=2024-10-01`
- **Authentication**: Power Platform delegated token
- **Required Permission**: Power Platform Administrator role
- **Source File**: `src/app/api/power-platform/environments/{environmentId}/enable-managed/route.js`
- **Response**: Success or error status
- **Documentation**: [Enable Managed Environment](https://learn.microsoft.com/en-us/power-platform/admin/managed-environment-enable)

### Disable Managed Environment
- **Endpoint**: `POST https://api.powerplatform.com/environmentmanagement/environments/{environmentId}/disableManagedEnvironment?api-version=2024-10-01`
- **Authentication**: Power Platform delegated token
- **Required Permission**: Power Platform Administrator role
- **Source File**: `src/app/api/power-platform/environments/{environmentId}/disable-managed/route.js`
- **Response**: Success or error status
- **Documentation**: [Managed Environment Management](https://learn.microsoft.com/en-us/power-platform/admin/managed-environment-enable)

### Delete Environment
- **Endpoint**: `DELETE https://api.powerplatform.com/environmentmanagement/environments/{environmentId}?api-version=2024-10-01`
- **Authentication**: Power Platform delegated token
- **Required Permission**: Power Platform Administrator role
- **Source File**: `src/app/api/power-platform/environments/{environmentId}/delete/route.js`
- **Response**: 202 Accepted with operation tracking
- **Documentation**: [Delete Environment](https://learn.microsoft.com/en-us/rest/api/power-platform/environmentmanagement/environments/delete-environment)

### Query Power Platform Inventory
- **Endpoint**: `POST https://api.powerplatform.com/resourcequery/resources/query?api-version=2024-10-01`
- **Authentication**: Power Platform delegated token (scope: `https://api.powerplatform.com/.default`)
- **Required Permission**: Power Platform Administrator role
- **Source File**: `src/app/api/power-platform/inventory/route.js`
- **Request Body**:
  ```json
  {
    "TableName": "PowerPlatformResources",
    "Clauses": [
      {
        "$type": "summarize",
        "SummarizeClauseExpression": {
          "OperatorName": "count",
          "OperatorFieldName": "resourceCount",
          "FieldList": ["type"]
        }
      },
      {
        "$type": "orderby",
        "FieldNamesAscDesc": { "resourceCount": "desc" }
      }
    ]
  }
  ```
- **Response**: JSON with `totalRecords`, `count`, `resultTruncated`, `data[]` array
- **Supported Clauses**: where, project, take, orderby, distinct, count, summarize, extend, join
- **Documentation**: [Power Platform Inventory API](https://learn.microsoft.com/en-us/power-platform/admin/inventory-api)

### Send Data to Power Automate Flow
- **Endpoint**: `POST /api/power-platform/send-to-flow` (internal proxy)
- **Target**: User-provided Power Automate HTTP trigger URL
- **Authentication**: 
  - Detects SAS authentication in URL (sig, sp, sv, se parameters)
  - If no SAS: Acquires Power Automate delegated token (`https://service.flow.microsoft.com/.default`)
- **Required Permission**: Depends on flow configuration
- **Source File**: `src/app/api/power-platform/send-to-flow/route.js`
- **Request Body**: 
  ```json
  {
    "flowUrl": "https://prod-XX.eastus.logic.azure.com:443/workflows/.../triggers/manual/paths/invoke?...",
    "payload": { /* arbitrary JSON data */ }
  }
  ```
- **Response**: Proxied response from Power Automate flow
- **Note**: Handles both "Anyone" (SAS) and "Any authenticated user in tenant" (OAuth) flow authentication
- **Documentation**: [Power Automate HTTP triggers](https://learn.microsoft.com/en-us/power-automate/triggers-introduction)

---

## External Data Sources

### National Vulnerability Database (NVD)
- **Endpoint**: `GET https://services.nvd.nist.gov/rest/json/cves/2.0`
- **Authentication**: None (optional API key for higher rate limits)
- **Source File**: `src/app/api/nvd/route.js`
- **Query Parameters**:
  - `resultsPerPage`: Limit (max 2000)
  - `startIndex`: Pagination offset
  - `pubStartDate`, `pubEndDate`: Publication date range
  - `lastModStartDate`, `lastModEndDate`: Modification date range
  - `cvssV3Severity`: e.g., CRITICAL, HIGH
- **Response**: JSON with CVE list
- **Rate Limit**: 5 requests per 30 seconds (10/30s with API key)
- **Documentation**: [NVD API 2.0](https://nvd.nist.gov/developers/vulnerabilities)

### Microsoft Security Response Center (MSRC)
- **Endpoint**: `GET https://api.msrc.microsoft.com/cvrf/v2.0/cvrf/{id}`
- **Authentication**: None
- **Source File**: `src/app/api/msrc/route.js`
- **Path Parameters**: `{id}` - CVRF document ID (e.g., 2025-Jan)
- **Response**: XML (CVRF format) with security updates
- **Documentation**: [MSRC CVRF API](https://api.msrc.microsoft.com/cvrf/v2.0/swagger/index)

### Common Weakness Enumeration (CWE)
- **Endpoint**: `GET https://cwe.mitre.org/data/xml/cwec_latest.xml.zip`
- **Authentication**: None
- **Source File**: `src/app/api/cwe/route.js`
- **Response**: ZIP archive containing CWE XML database
- **Documentation**: [CWE List](https://cwe.mitre.org/data/index.html)

### Windows Update Catalog
- **Endpoint**: `https://www.catalog.update.microsoft.com/Search.aspx`
- **Authentication**: None
- **Source File**: `src/app/api/windows-updates/catalog/route.js`
- **Method**: Web scraping (HTML parsing)
- **Query Parameters**: `q` - Search query (e.g., KB number)
- **Response**: HTML (parsed for update information)
- **Note**: Not an official API; subject to breaking changes

---

## Rate Limits Summary

| API | Rate Limit | Retry Header | Documentation |
|-----|------------|--------------|---------------|
| **Microsoft Graph** | Varies (1k-10k req/min per app) | `Retry-After` | [Throttling guidance](https://learn.microsoft.com/en-us/graph/throttling) |
| **Azure Resource Manager** | 12,000 req/hour per subscription | `Retry-After` | [Request limits](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/request-limits-and-throttling) |
| **Dataverse** | 6,000 req per 5 min per user | `Retry-After` | [Service protection limits](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/api-limits) |
| **Power Platform API** | Inherited from Dataverse | `Retry-After` | Same as Dataverse |
| **NVD** | 5 req per 30 sec (10 with key) | None | [NVD API docs](https://nvd.nist.gov/developers/start-here) |

---

## Common Headers Used

### Microsoft Graph & Dataverse OData Headers
```http
Authorization: Bearer {token}
Accept: application/json
Content-Type: application/json
OData-MaxVersion: 4.0
OData-Version: 4.0
Prefer: odata.include-annotations="*"
```

### Azure Resource Manager Headers
```http
Authorization: Bearer {token}
Content-Type: application/json
```

### Power Platform API Headers
```http
Authorization: Bearer {token}
Accept: application/json
```

---

**Document Version:** 1.0  
**Last Updated:** November 1, 2025  
**Maintained By:** Development Team

For detailed authentication and permission setup, refer to [API-DOCUMENTATION.md](./API-DOCUMENTATION.md).
