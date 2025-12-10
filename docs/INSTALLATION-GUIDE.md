# Pulse 360° - Complete Installation Guide

This guide provides step-by-step instructions to install and configure Pulse 360° in your Azure tenant. For MSPs, this includes instructions for managing multiple client tenants from a single portal.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Entra ID App Registration](#entra-id-app-registration)
- [Environment Configuration](#environment-configuration)
- [Installation Steps](#installation-steps)
- [Multi-Tenant Configuration for MSPs](#multi-tenant-configuration-for-msps)
- [Azure Key Vault Setup (Recommended)](#azure-key-vault-setup-recommended)
- [Power Platform Configuration](#power-platform-configuration)
- [Verification & Testing](#verification--testing)
- [Troubleshooting](#troubleshooting)

---

## Prerequisites

Before starting, ensure you have:

- **Node.js 18+** installed ([Download](https://nodejs.org/))
- **Azure Subscription** with Global Administrator or Application Administrator role
- **Power Platform Environment** (optional, for Power Platform features)
- **Git** installed ([Download](https://git-scm.com/))
- **Visual Studio Code** (recommended) ([Download](https://code.visualstudio.com/))

### Required Azure Permissions

- **Application Administrator** or **Global Administrator** to create app registrations
- **Privileged Role Administrator** to assign admin roles
- **User Administrator** to manage users and groups

---

## Entra ID App Registration

### Step 1: Create the App Registration

1. Navigate to [Azure Portal](https://portal.azure.com)
2. Go to **Microsoft Entra ID** → **App registrations**
3. Click **+ New registration**

**Configuration:**
- **Name**: `Pulse 360 Admin Portal` (or your preferred name)
- **Supported account types**: 
  - For single tenant: **Accounts in this organizational directory only**
  - For MSP multi-tenant: **Accounts in any organizational directory (Any Microsoft Entra ID tenant - Multitenant)**
- **Redirect URI**: 
  - Platform: **Web**
  - URL: `http://localhost:3000/api/auth/callback/azure-ad` (for development)
  - For production: `https://your-domain.com/api/auth/callback/azure-ad`

4. Click **Register**

### Step 2: Note Your Application Details

After registration, note the following from the **Overview** page:
- **Application (client) ID**: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`
- **Directory (tenant) ID**: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

### Step 3: Create Client Secret

1. In your app registration, go to **Certificates & secrets**
2. Click **+ New client secret**
3. **Description**: `Pulse 360 Production Secret`
4. **Expires**: Choose based on your security policy (recommended: 24 months)
5. Click **Add**
6. **IMPORTANT**: Copy the secret **Value** immediately (you cannot view it again)
   - Secret Value: `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

### Step 4: Configure API Permissions

Navigate to **API permissions** and add the following:

#### Microsoft Graph Delegated Permissions

Click **+ Add a permission** → **Microsoft Graph** → **Delegated permissions**

**User & Identity:**
- `User.Read` - Sign in and read user profile
- `User.ReadWrite.All` - Read and write all users' full profiles
- `Directory.ReadWrite.All` - Read and write directory data

**Groups:**
- `Group.ReadWrite.All` - Read and write all groups
- `GroupMember.ReadWrite.All` - Read and write all group memberships

**Applications:**
- `Application.ReadWrite.All` - Read and write all applications

**Security:**
- `SecurityEvents.ReadWrite.All` - Read and write security events
- `IdentityRiskEvent.Read.All` - Read identity risk event information
- `IdentityRiskyUser.Read.All` - Read risky user information

**Devices & Intune:**
- `Device.Read.All` - Read all devices
- `DeviceManagementConfiguration.Read.All` - Read Microsoft Intune device configuration
- `DeviceManagementManagedDevices.Read.All` - Read Microsoft Intune devices
- `DeviceManagementApps.Read.All` - Read Microsoft Intune apps

**Azure & Subscriptions:**
- `offline_access` - Maintain access to data you have given it access to
- `openid` - Sign users in
- `profile` - View users' basic profile
- `email` - View users' email address

**Audit & Reports:**
- `AuditLog.Read.All` - Read audit log data
- `Reports.Read.All` - Read all usage reports

#### Microsoft Graph Application Permissions (Optional - for automation)

Click **+ Add a permission** → **Microsoft Graph** → **Application permissions**

- `User.Read.All` - Read all users' full profiles
- `Directory.Read.All` - Read directory data
- `Group.Read.All` - Read all groups
- `Application.Read.All` - Read all applications

#### Power Platform API (Optional - for Power Platform features)

1. Click **+ Add a permission**
2. Select **APIs my organization uses**
3. Search for: **Power Automate Service** or **PowerApps**
4. Select **Delegated permissions**
5. Add:
   - `Flows.Manage.All` - Manage all flows

**Note**: Power Platform API may not appear until at least one user in your tenant has signed into https://make.powerautomate.com

### Step 5: Grant Admin Consent

1. Click **✓ Grant admin consent for [Your Organization]**
2. Click **Yes** to confirm
3. Wait for green checkmarks to appear next to all permissions

---

## Environment Configuration

### Step 1: Clone the Repository

```bash
git clone https://github.com/russrimm/PortalofPortals.git
cd PortalofPortals
```

### Step 2: Install Dependencies

```bash
npm install
```

### Step 3: Create Environment File

Create a file named `.env.local` in the root directory:

```env
# NextAuth Configuration
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=your-random-secret-here-generate-with-openssl

# Azure AD Configuration - Primary Tenant
AZURE_TENANT_ID=your-tenant-id-here
AZURE_CLIENT_ID=your-client-id-here
AZURE_CLIENT_SECRET=your-client-secret-here

# Azure Subscription (Optional - for Azure Resource features)
AZURE_SUBSCRIPTION_ID=your-subscription-id-here

# Multi-Tenant Configuration (Optional - for MSPs)
# Add additional tenants as comma-separated values
ADDITIONAL_TENANT_IDS=tenant-id-2,tenant-id-3
ADDITIONAL_TENANT_NAMES=Client Tenant 1,Client Tenant 2

# Azure Key Vault (Recommended for Production)
AZURE_KEY_VAULT_NAME=your-keyvault-name
USE_MANAGED_IDENTITY=false

# Feature Flags
ENABLE_POWER_PLATFORM=true
ENABLE_INTUNE=true
ENABLE_AZURE_RESOURCES=true

# Logging & Debugging
ENABLE_API_LOGGING=false
NODE_ENV=development
```

### Step 4: Generate Secure Secret

Generate a secure secret for NextAuth:

**Windows (PowerShell):**
```powershell
node -e "console.log(require('crypto').randomBytes(32).toString('base64'))"
```

**Mac/Linux:**
```bash
openssl rand -base64 32
```

Copy the output and set it as `NEXTAUTH_SECRET` in your `.env.local` file.

---

## Installation Steps

### Step 1: Run Development Server

```bash
npm run dev
```

The application will start on [http://localhost:3000](http://localhost:3000)

### Step 2: First-Time Login

1. Navigate to [http://localhost:3000](http://localhost:3000)
2. Click **Sign In** or navigate to any protected page
3. You will be redirected to Microsoft login
4. Sign in with your Azure AD credentials
5. **Consent** to the requested permissions
6. You will be redirected back to the portal

### Step 3: Verify Access

After logging in, verify you can access:
- **Dashboard** - Main navigation
- **Users** - Microsoft 365 users list
- **Groups** - Azure AD groups
- **Devices** - Intune-managed devices (if enabled)
- **Azure Resources** - Azure subscription resources (if subscription ID configured)
- **Power Platform** - Environments and flows (if Power Platform permissions granted)

---

## Multi-Tenant Configuration for MSPs

Pulse 360° supports managing multiple client tenants from a single portal. This is ideal for Managed Service Providers (MSPs).

### Architecture Overview

```
┌─────────────────────────────────────────┐
│       Pulse 360° Portal (MSP)           │
│  ┌───────────────────────────────────┐  │
│  │  Tenant Selector Dropdown         │  │
│  │  - Primary Tenant                 │  │
│  │  - Client Tenant 1                │  │
│  │  - Client Tenant 2                │  │
│  └───────────────────────────────────┘  │
│                                         │
│  Current Tenant: Client Tenant 1       │
│  ┌───────────────────────────────────┐  │
│  │  Users, Groups, Devices, etc.    │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

### Step 1: Enable Multi-Tenant in App Registration

1. Go to your App Registration in Azure Portal
2. Navigate to **Authentication**
3. Under **Supported account types**, select:
   - **Accounts in any organizational directory (Any Microsoft Entra ID tenant - Multitenant)**
4. Click **Save**

### Step 2: Configure Additional Tenants

Add to your `.env.local`:

```env
# Multi-Tenant Configuration
ADDITIONAL_TENANT_IDS=00000000-0000-0000-0000-000000000001,00000000-0000-0000-0000-000000000002
ADDITIONAL_TENANT_NAMES=Contoso Ltd,Fabrikam Inc
```

### Step 3: Client Tenant Consent

For each client tenant, an administrator must grant consent:

1. Generate consent URL for each tenant:
   ```
   https://login.microsoftonline.com/{TENANT_ID}/adminconsent?client_id={CLIENT_ID}&redirect_uri={REDIRECT_URI}
   ```

2. Replace:
   - `{TENANT_ID}` with the client's tenant ID
   - `{CLIENT_ID}` with your app's client ID
   - `{REDIRECT_URI}` with your app's redirect URI (URL-encoded)

3. Send this URL to the client's Global Administrator
4. Client admin clicks the link and grants consent

**Example:**
```
https://login.microsoftonline.com/client-tenant-id-here/adminconsent?client_id=your-app-id-here&redirect_uri=https%3A%2F%2Fyour-domain.com%2Fapi%2Fauth%2Fcallback%2Fazure-ad
```

### Step 4: Implement Tenant Switcher

The portal will include a tenant selector in the navigation bar. When you switch tenants:
- All API calls use the selected tenant's credentials
- Data displayed is for the selected tenant only
- The selected tenant persists in session storage

---

## Azure Key Vault Setup (Recommended)

For production deployments, store secrets in Azure Key Vault and use Managed Identity for secure access.

### Step 1: Create Key Vault

```bash
az keyvault create \
  --name pulse360-secrets \
  --resource-group pulse360-rg \
  --location eastus \
  --enable-rbac-authorization true
```

### Step 2: Store Secrets

```bash
# Primary tenant secrets
az keyvault secret set --vault-name pulse360-secrets --name "AzureClientId" --value "your-client-id"
az keyvault secret set --vault-name pulse360-secrets --name "AzureClientSecret" --value "your-client-secret"
az keyvault secret set --vault-name pulse360-secrets --name "AzureTenantId" --value "your-tenant-id"

# Additional tenant secrets (for MSPs)
az keyvault secret set --vault-name pulse360-secrets --name "ClientTenant1-ClientId" --value "client-1-app-id"
az keyvault secret set --vault-name pulse360-secrets --name "ClientTenant1-ClientSecret" --value "client-1-secret"
```

### Step 3: Create Managed Identity

```bash
# Create user-assigned managed identity
az identity create \
  --name pulse360-identity \
  --resource-group pulse360-rg \
  --location eastus

# Get the identity principal ID
PRINCIPAL_ID=$(az identity show \
  --name pulse360-identity \
  --resource-group pulse360-rg \
  --query principalId -o tsv)

# Grant Key Vault Secrets User role
az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee $PRINCIPAL_ID \
  --scope /subscriptions/{subscription-id}/resourceGroups/pulse360-rg/providers/Microsoft.KeyVault/vaults/pulse360-secrets
```

### Step 4: Configure Application

Update `.env.local` for production:

```env
AZURE_KEY_VAULT_NAME=pulse360-secrets
USE_MANAGED_IDENTITY=true
AZURE_MANAGED_IDENTITY_CLIENT_ID=your-managed-identity-client-id
```

---

## Power Platform Configuration

### Enable Power Platform Features

1. Ensure Power Platform API permissions are granted (see Step 4 in App Registration)
2. Set `ENABLE_POWER_PLATFORM=true` in `.env.local`
3. Restart the development server

### Export & Deploy Solutions

The portal supports exporting and deploying Power Platform solutions:

**Features:**
- Export solutions as **Managed** or **Unmanaged**
- Select target environment from dropdown
- Automatic deployment with authentication
- HTTP trigger configuration for "Any user in tenant" or "Specific users"

**Workflow:**
1. Navigate to **Power Platform** → **Environments**
2. Select source environment
3. Click **Export Solution**
4. Choose solution and export type
5. Select target environment
6. Configure authentication requirements
7. Click **Deploy**

### Create Dataverse Application User

For automated flows and system-level operations:

1. Navigate to **Power Platform** → **Create Application User**
2. Enter Application (Client) ID from Azure AD
3. Select Dataverse environment
4. Assign security role
5. Click **Create**

This creates a non-interactive application user that can run flows and access Dataverse.

---

## Verification & Testing

### Test Basic Functionality

1. **Authentication**: Sign in and verify token refresh
2. **Users Page**: View Microsoft 365 users
3. **Groups Page**: View and search Azure AD groups
4. **Devices Page**: View Intune-managed devices
5. **Azure Resources**: Query Azure resources with KQL
6. **Power Platform**: View environments and flows

### Test Multi-Tenant (MSP)

1. Switch tenant using dropdown
2. Verify users list updates for selected tenant
3. Check that resources belong to selected tenant
4. Switch back to primary tenant

### Test Solution Export

1. Navigate to Power Platform → Environments
2. Select environment with solutions
3. Export a test solution
4. Deploy to another environment
5. Verify deployment in target environment

---

## Troubleshooting

### Common Issues

#### 1. "Not authenticated" or 401 errors

**Cause**: Missing or expired Azure AD credentials

**Solution:**
- Verify `.env.local` has correct `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, `AZURE_TENANT_ID`
- Check that client secret hasn't expired
- Sign out and sign in again

#### 2. "Power Platform API returned 500"

**Cause**: Missing Power Platform API permissions

**Solution:**
- Add Power Automate Service API permissions (see Step 4)
- Grant admin consent
- Have at least one user sign into https://make.powerautomate.com
- Sign out and sign in again to get new token

#### 3. "Failed to acquire Power Platform token"

**Cause**: Token audience mismatch or missing OBO permissions

**Solution:**
- Verify `offline_access` delegated permission is granted
- Check that initial login scope doesn't include Sites.Read.All
- Review `src/lib/auth.ts` scope configuration

#### 4. Multi-tenant consent not working

**Cause**: App not configured as multi-tenant or consent URL incorrect

**Solution:**
- Verify app registration is set to "Multitenant"
- Double-check consent URL format
- Ensure client admin has Global Administrator role

#### 5. Azure Key Vault access denied

**Cause**: Managed Identity doesn't have Key Vault access

**Solution:**
- Verify role assignment: `Key Vault Secrets User`
- Check managed identity is assigned to App Service
- Wait 5-10 minutes for role assignment to propagate

### Enable Debug Logging

Set in `.env.local`:
```env
ENABLE_API_LOGGING=true
NODE_ENV=development
```

View logs in terminal where `npm run dev` is running.

---

## Production Deployment

### Azure App Service Deployment

1. **Create App Service:**
   ```bash
   az webapp create \
     --name pulse360 \
     --resource-group pulse360-rg \
     --plan pulse360-plan \
     --runtime "NODE:18-lts"
   ```

2. **Configure Environment Variables:**
   ```bash
   az webapp config appsettings set \
     --name pulse360 \
     --resource-group pulse360-rg \
     --settings \
       NEXTAUTH_URL=https://pulse360.azurewebsites.net \
       NEXTAUTH_SECRET=your-production-secret \
       AZURE_KEY_VAULT_NAME=pulse360-secrets \
       USE_MANAGED_IDENTITY=true
   ```

3. **Assign Managed Identity:**
   ```bash
   az webapp identity assign \
     --name pulse360 \
     --resource-group pulse360-rg
   ```

4. **Deploy:**
   ```bash
   npm run build
   az webapp deployment source config-zip \
     --name pulse360 \
     --resource-group pulse360-rg \
     --src ./build.zip
   ```

### Update Redirect URIs

After deployment, update your Azure AD App Registration:
1. Go to **Authentication**
2. Add production redirect URI:
   - `https://pulse360.azurewebsites.net/api/auth/callback/azure-ad`
3. Update `NEXTAUTH_URL` in App Service configuration

---

## Next Steps

- **Configure Alerts**: Set up Azure Monitor alerts for errors
- **Enable Application Insights**: Monitor performance and usage
- **Set Up Backups**: Configure automated backups for Key Vault
- **Review Security**: Enable Microsoft Defender for Cloud
- **Documentation**: Create runbooks for your team
- **Training**: Train MSP staff on tenant switching and solution deployment

---

## Support & Resources

- **Documentation**: [Full Documentation](./README.md)
- **API Reference**: [API Documentation](./API-DOCUMENTATION.md)
- **Feature Guide**: [Feature Walkthrough](./FEATURE-WALKTHROUGH.md)
- **GitHub Issues**: [Report Issues](https://github.com/russrimm/PortalofPortals/issues)

---

**Last Updated**: November 22, 2025  
**Version**: 1.0.0
