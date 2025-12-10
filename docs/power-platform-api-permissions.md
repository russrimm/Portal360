# Power Platform API Permissions Setup

This document describes how to configure the required API permissions for Power Platform features in Pulse 360°.

## Overview

Power Platform APIs require specific delegated permissions to be granted to your Azure AD application registration. Without these permissions, features like **Tenant Applications** and some **Environment Management** capabilities will fail with consent errors.

## Required API Permissions

### PowerApps-Advisor API (BAP API)

The Business Application Platform (BAP) API is used for:
- Tenant-wide application management
- Environment administrative operations
- DLP policy management

**API Details:**
- **API Name**: PowerApps-Advisor
- **Application ID**: `8578e004-a5c6-46e7-913e-12f58912df43`
- **Resource URI**: `https://api.bap.microsoft.com/`
- **Required Permission**: `user_impersonation` (Delegated)

## Setup Instructions

### Method 1: Azure Portal (Recommended)

1. **Navigate to App Registration**
   - Go to [Azure Portal](https://portal.azure.com)
   - Navigate to **Azure Active Directory** > **App registrations**
   - Select your app registration (e.g., "Pulse360-AdminPortal")

2. **Add API Permission**
   - Click **API permissions** in the left menu
   - Click **Add a permission**
   - Click **APIs my organization uses** tab
   - Search for **"PowerApps-Advisor"** or paste the Application ID: `8578e004-a5c6-46e7-913e-12f58912df43`
   - Click on the result

3. **Select Permission Type**
   - Click **Delegated permissions**
   - Check the box for **user_impersonation**
   - Click **Add permissions**

4. **Grant Admin Consent**
   - Back on the API permissions page, click **Grant admin consent for [Your Tenant]**
   - Click **Yes** to confirm
   - Wait for the green checkmark to appear in the "Status" column

5. **Verify**
   - You should see the permission listed as:
     - **API**: PowerApps-Advisor
     - **Permission**: user_impersonation
     - **Type**: Delegated
     - **Status**: ✅ Granted for [Your Tenant]

### Method 2: Admin Consent URL

You can also grant consent directly via URL:

```
https://login.microsoftonline.com/{TENANT_ID}/adminconsent?client_id={CLIENT_ID}
```

Replace:
- `{TENANT_ID}` with your Azure AD tenant ID
- `{CLIENT_ID}` with your application's client ID

**Example:**
```
https://login.microsoftonline.com/contoso.onmicrosoft.com/adminconsent?client_id=32c72b33-a331-4545-ad5a-6d2fdf961c4c
```

### Method 3: PowerShell

```powershell
# Install Microsoft Graph PowerShell if not already installed
Install-Module Microsoft.Graph -Scope CurrentUser

# Connect to Microsoft Graph
Connect-MgGraph -Scopes "Application.ReadWrite.All", "AppRoleAssignment.ReadWrite.All"

# Get your app registration
$appId = "YOUR_CLIENT_ID_HERE"
$app = Get-MgApplication -Filter "appId eq '$appId'"

# Get PowerApps-Advisor service principal
$bapServicePrincipal = Get-MgServicePrincipal -Filter "appId eq '8578e004-a5c6-46e7-913e-12f58912df43'"

# Get the user_impersonation scope ID
$scope = $bapServicePrincipal.Oauth2PermissionScopes | Where-Object { $_.Value -eq "user_impersonation" }

# Add the required resource access
$requiredResourceAccess = @{
    ResourceAppId = "8578e004-a5c6-46e7-913e-12f58912df43"
    ResourceAccess = @(
        @{
            Id = $scope.Id
            Type = "Scope"
        }
    )
}

# Update the app registration
Update-MgApplication -ApplicationId $app.Id -RequiredResourceAccess $requiredResourceAccess

# Grant admin consent (requires admin)
New-MgServicePrincipalAppRoleAssignment `
    -ServicePrincipalId $app.Id `
    -PrincipalId $app.Id `
    -ResourceId $bapServicePrincipal.Id `
    -AppRoleId $scope.Id
```

## Troubleshooting

### Error: "User or administrator has not consented"

**Full Error:**
```
AADSTS65001: The user or administrator has not consented to use the application 
with ID 'YOUR_APP_ID' named 'YOUR_APP_NAME'. Send an interactive authorization 
request for this user and resource.
```

**Solution:**
1. Follow the setup instructions above to add the API permission
2. Grant admin consent
3. Sign out and sign in again to get a fresh token with the new scopes

### Error: "Invalid audience"

**Full Error:**
```
The received access token has been obtained from wrong audience or resource 
'https://api.powerplatform.com'. It should exactly match with one of the 
allowed audiences 'https://api.bap.microsoft.com/'
```

**Solution:**
This error indicates the wrong token function is being used. Ensure:
- Tenant Applications uses `getBAPDelegatedToken`
- Regular Power Platform operations use `getPowerPlatformDelegatedToken`
- Flow operations use `getPowerAutomateDelegatedToken`

### Permission not appearing in Azure Portal

If you can't find "PowerApps-Advisor" when searching:

1. Try searching by Application ID: `8578e004-a5c6-46e7-913e-12f58912df43`
2. Ensure you're in the **"APIs my organization uses"** tab, not "Microsoft APIs"
3. Verify your tenant has Power Platform enabled
4. Try the admin consent URL method instead

### Consent granted but still failing

1. **Sign out completely** from the application
2. **Clear browser cookies/cache** (or use incognito mode)
3. **Sign in again** to get a fresh token with the new permissions
4. Check the token acquisition logs in the console for specific error messages

## Testing

After granting consent, test the following features:

1. **Tenant Applications**
   - Navigate to: Application Management → Tenant Applications
   - Should load without 401/403 errors
   - Should display Power Platform apps across the tenant

2. **Flow Runs**
   - Navigate to: Power Platform → Environments → [Environment] → Flows → [Flow]
   - Click "View Flow Runs"
   - Should display flow execution history

3. **Tenant Settings**
   - Navigate to: Power Platform → Tenant Settings
   - Should display tenant-wide Power Platform configuration

## References

- [Power Platform Admin API Overview](https://learn.microsoft.com/en-us/power-platform/admin/programmability-overview)
- [Authentication for Power Platform APIs](https://learn.microsoft.com/en-us/power-platform/admin/programmability-authentication-v2)
- [Azure AD Consent Framework](https://learn.microsoft.com/en-us/azure/active-directory/develop/application-consent-experience)
- [Microsoft Graph App Consent](https://learn.microsoft.com/en-us/graph/auth-v2-user)

## Support

If you continue to experience issues after following these steps:

1. Check the browser console for detailed error messages
2. Verify your user account has Power Platform admin roles
3. Ensure the app registration has the correct redirect URIs configured
4. Review the [API Documentation](./API-DOCUMENTATION.md) for additional setup requirements
