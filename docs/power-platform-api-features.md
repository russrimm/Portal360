# Power Platform API Features Implementation

## Overview
This document describes the Power Platform REST API features exposed in the Pulse 360° application, based on the official Microsoft Power Platform API documentation.

## API Endpoint
- **Base URL**: `https://api.bap.microsoft.com`
- **API Version**: `2020-10-01`
- **Authentication**: OAuth 2.0 with client credentials (service principal)
- **Scope**: `https://api.bap.microsoft.com/.default`

## Environment Properties Exposed

### Basic Information
- **Environment Name** (`displayName`)
- **Environment ID** (`id`, `name`)
- **Location/Region** (`location`, `azureRegion`)
- **Environment Type** (`environmentSku`): Production, Sandbox, Trial, Developer, Teams, etc.
- **Database Type** (`databaseType`): Dataverse, None, etc.
- **Creation Time** (`createdTime`)
- **Created By** (`createdBy.displayName`, `createdBy.id`)
- **Provisioning State** (`provisioningState`)

### Governance & Protection Features ✨ NEW

#### 1. **Managed Environment Status**
- **Property**: `properties.governanceConfiguration.protectionLevel`
- **Display**: Green "🛡️ Managed" badge when value is `"Standard"`
- **Values**: 
  - `"Standard"` = Managed Environment (Enhanced Governance)
  - `"Basic"` = Not a Managed Environment
- **Purpose**: Indicates environment has enhanced governance capabilities
- **Features Enabled**:
  - Environment groups
  - Sharing limits
  - Usage insights
  - Data policies enforcement
  - Solution checker integration
  - IP firewall
  - Extended backup (28 days)
  - Maker welcome content

#### 2. **Customer-Managed Key (CMK)**
- **Property**: `properties.protectionStatus.keyManagedBy`
- **Display**: Purple "🔐 CMK" badge when set to "Customer"
- **Purpose**: Indicates environment uses customer-managed encryption keys
- **Benefits**: Enhanced data sovereignty and compliance control

#### 3. **Release Cycle**
- **Property**: `properties.cluster.category`
- **Display**: Blue "🔄 {cycle}" badge
- **Purpose**: Shows which release cycle (update wave) the environment is on
- **Values**:
  - `"FirstRelease"` = Early (First Release) - Gets updates first
  - `"Prod"` = Standard - Gets updates on standard schedule
- **Additional Context**: Cluster number (`cluster.number`) and geo (`cluster.geoShortName`)

#### 4. **Backup Retention**
- **Property**: `retentionDetails.retentionPeriod`
- **Additional**: `retentionDetails.backupsAvailableFromDateTime`
- **Display**: Shown in Governance & Protection section
- **Purpose**: Backup retention policy (7 days default, 28 days for Managed Environments)

### Connected Microsoft 365 Groups ✨ NEW
- **Property**: `connectedGroups`
- **Type**: Array of group objects
- **Display**: Separate section listing connected groups with IDs
- **Purpose**: Shows which Microsoft 365 groups have access to the environment

### Storage & Capacity
- **Database Storage**: `capacity[].actualConsumption` (where `capacityType` = "Database")
- **File Storage**: `capacity[].actualConsumption` (where `capacityType` = "File")
- **Log Storage**: `capacity[].actualConsumption` (where `capacityType` = "Log")
- **Unit**: `capacity[].capacityUnit` (typically "MB")
- **Last Updated**: `capacity[].updatedOn`

### Dataverse Organization Details
- **Instance URL**: `linkedEnvironmentMetadata.instanceUrl`
- **API URL**: `linkedEnvironmentMetadata.instanceApiUrl`
- **Domain Name**: `linkedEnvironmentMetadata.domainName`
- **Unique Name**: `linkedEnvironmentMetadata.uniqueName`
- **Version**: `linkedEnvironmentMetadata.version`
- **State**: `linkedEnvironmentMetadata.instanceState`
- **Base Language**: `linkedEnvironmentMetadata.baseLanguage`

### Runtime Endpoints
- **Property**: `runtimeEndpoints`
- **Services**: 
  - `microsoft.Flow` - Power Automate
  - `microsoft.PowerApps` - Power Apps
  - `microsoft.BusinessAppPlatform` - BAP
  - `microsoft.CommonDataModel` - CDM
  - `microsoft.ApiManagement` - APIM
  - `microsoft.PowerAppsAdvisor` - Advisor

## UI Enhancements

### Badge System
Compact badges displayed on collapsed environment cards:
1. **Environment Type** (e.g., "Production", "Sandbox") - White badge
2. **Managed Environment** - Green badge with shield icon
3. **Customer-Managed Key** - Purple badge with lock icon
4. **Update Ring** - Blue badge with refresh icon

### Governance & Protection Section
New dedicated section in expanded environment details showing:
- Managed Environment status with feature description
- Encryption key management type
- Backup retention period and available backups timeline
- Update ring assignment

### Connected Groups Display
Separate section listing all connected Microsoft 365 groups with:
- Group display name
- Group ID (for reference)

## API Query Patterns

### List All Environments (Full Details)
```
GET https://api.bap.microsoft.com/providers/Microsoft.BusinessAppPlatform/scopes/admin/environments?api-version=2020-10-01&$expand=properties.capacity,properties.addons,properties.linkedEnvironmentMetadata
```

### Get Single Environment
```
GET https://api.bap.microsoft.com/providers/Microsoft.BusinessAppPlatform/scopes/admin/environments/{environmentId}?api-version=2020-10-01&$expand=properties.capacity,properties.addons,properties.linkedEnvironmentMetadata
```

### List Environments (Summary - Minimal)
```
GET https://api.bap.microsoft.com/providers/Microsoft.BusinessAppPlatform/scopes/admin/environments?api-version=2020-10-01
```
*Note: Summary mode strips properties to minimal set: id, name, location, type, displayName*

## Caching Strategy
- **List requests**: 60 seconds TTL
- **Individual environment details**: 120 seconds TTL
- **Token**: Cached until 60 seconds before expiry

## Related APIs (Available but Not Yet Implemented)

### Tenant Settings API
- **Endpoint**: `POST https://api.bap.microsoft.com/providers/Microsoft.BusinessAppPlatform/listtenantsettings?api-version=2020-10-01`
- **Returns**: Tenant-wide governance settings including:
  - `powerPlatform.governance.disableAdminDigest` - Managed Environment admin digest
  - `powerPlatform.licensing.ApplyAutoClaimToOnlyManagedEnvironments` - License autoclaim settings
  - Environment creation restrictions
  - Copilot settings
  - Sharing policies

### App Management API
- **Base**: `https://api.powerplatform.com/appmanagement`
- **Purpose**: Manage first-party app installations (Dynamics 365, etc.)
- **Operations**: Install, update, list application packages

### Licensing/Billing Policies API
- **Base**: `https://api.powerplatform.com/licensing`
- **Purpose**: Manage billing policies and capacity allocation
- **Operations**: Create, read, update billing policies

## PowerShell Equivalent
For reference, the PowerShell cmdlet that exposes these features:
```powershell
Get-AdminPowerAppEnvironment -EnvironmentName {guid} -GetProtectedEnvironment
```

## References
- [Power Platform API Reference](https://learn.microsoft.com/en-us/rest/api/power-platform/)
- [Tutorial: Create Daily Capacity Report](https://learn.microsoft.com/en-us/power-platform/admin/programmability-tutorial-create-daily-capacity-report)
- [List Tenant Settings](https://learn.microsoft.com/en-us/power-platform/admin/list-tenantsettings)
- [Managed Environments Overview](https://learn.microsoft.com/en-us/power-platform/admin/managed-environment-overview)
- [PowerShell for Power Platform](https://learn.microsoft.com/en-us/power-platform/admin/powerapps-powershell)

## Future Enhancements
Potential additional features from the API:
1. **Environment Operations API**: Backup, restore, copy, reset operations
2. **Tenant Settings Management**: Read/write tenant-wide governance policies
3. **Billing Policy Management**: Capacity allocation and billing policy CRUD
4. **Application Management**: First-party app installation status
5. **Environment Groups**: Group management for Managed Environments
6. **Data Policies**: DLP policy enforcement status per environment
7. **Usage Analytics**: API call metrics, storage trends over time

## Implementation Notes
- All properties are optional - UI gracefully handles missing data
- Badge display is conditional on property existence
- Managed Environment detection uses `states.management.id` presence
- CMK detection checks `protectionStatus.keyManagedBy === 'Customer'`
- Connected groups array may be empty even when property exists
- Storage values are in MB, converted to appropriate units for display
