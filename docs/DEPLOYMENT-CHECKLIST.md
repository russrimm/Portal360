# Production Deployment Checklist for IT Administrators

**Pre-deployment verification checklist for Pulse 360° Admin Portal**  
**Last Updated:** November 1, 2025

Use this checklist to ensure your Pulse 360° deployment is secure, properly configured, and ready for production use.

---

## Table of Contents

1. [Pre-Deployment Overview](#pre-deployment-overview)
2. [Azure App Registration Configuration](#azure-app-registration-configuration)
3. [API Permissions & Consent](#api-permissions--consent)
4. [Azure Role Assignments](#azure-role-assignments)
5. [Power Platform Configuration](#power-platform-configuration)
6. [Security & Secrets Management](#security--secrets-management)
7. [Application Configuration](#application-configuration)
8. [Testing & Validation](#testing--validation)
9. [Monitoring & Logging](#monitoring--logging)
10. [Production Deployment](#production-deployment)
11. [Post-Deployment Verification](#post-deployment-verification)
12. [Rollback Plan](#rollback-plan)

---

## Pre-Deployment Overview

### Timeline

**Recommended Timeline**: 2-4 weeks from start to production

| Phase | Duration | Activities |
|-------|----------|-----------|
| Planning | 2-3 days | Review requirements, obtain approvals |
| App Registration | 1 day | Create and configure Azure AD app |
| Permissions | 2-3 days | Grant permissions, wait for propagation |
| Testing (Dev) | 1 week | Functional testing in development |
| Testing (Staging) | 1 week | Full testing in staging environment |
| Production Deploy | 1 day | Deploy to production with validation |
| Monitoring | Ongoing | Monitor for issues and performance |

### Required Access

Ensure you have the following administrative access:

- [ ] **Global Administrator** or **Application Administrator** role in Microsoft Entra ID
- [ ] **Owner** or **Contributor** role on target Azure subscription
- [ ] **Power Platform Administrator** role
- [ ] Access to create **Azure Key Vault** and store secrets
- [ ] Access to production hosting environment (Azure App Service, Kubernetes, etc.)

### Documentation Review

Before starting, review:

- [ ] [API-DOCUMENTATION.md](./API-DOCUMENTATION.md) - Complete API reference
- [ ] [API-ENDPOINTS-REFERENCE.md](./API-ENDPOINTS-REFERENCE.md) - Endpoint quick reference
- [ ] [TROUBLESHOOTING.md](./TROUBLESHOOTING.md) - Common issues and solutions
- [ ] [Application Installation Guide](./application-installation.md) - Deployment steps

---

## Azure App Registration Configuration

### 1. Create App Registration

- [ ] Navigate to [Azure Portal](https://portal.azure.com) > Microsoft Entra ID > App registrations
- [ ] Click **New registration**
- [ ] Configure:
  - **Name**: `Pulse360-Production` (use environment-specific naming)
  - **Supported account types**: Single tenant
  - **Redirect URI**: 
    - Type: Web
    - URI: `https://your-production-domain.com/api/auth/callback/azure-ad`
- [ ] Click **Register**
- [ ] **Copy and save** the following values:
  - Application (client) ID → `AZURE_CLIENT_ID`
  - Directory (tenant) ID → `AZURE_TENANT_ID`

**Screenshot Location**: Save screenshot of app registration overview for documentation

### 2. Create Client Secret

- [ ] In app registration, go to **Certificates & secrets**
- [ ] Click **New client secret**
- [ ] Configure:
  - **Description**: `Pulse360-Production-Secret-2025`
  - **Expires**: 24 months (recommended)
- [ ] Click **Add**
- [ ] **⚠️ IMMEDIATELY COPY THE SECRET VALUE** → `AZURE_CLIENT_SECRET`
  - **This is the ONLY time you'll see the secret value**
  - Store securely in Azure Key Vault (see Security section)

**Security Note**: Never store this secret in code, config files, or documentation. Only store in secure secret management system.

### 3. Configure Authentication

- [ ] In app registration, go to **Authentication**
- [ ] Under **Platform configurations**, verify Web platform shows:
  - Redirect URI: `https://your-production-domain.com/api/auth/callback/azure-ad`
- [ ] Click **Add URI** if you need additional environments:
  - Staging: `https://staging.your-domain.com/api/auth/callback/azure-ad`
  - Development: `http://localhost:3000/api/auth/callback/azure-ad` (optional)
- [ ] Under **Implicit grant and hybrid flows**:
  - ✅ **ID tokens** (for implicit and hybrid flows)
  - ❌ Access tokens (not needed)
- [ ] Under **Advanced settings**:
  - **Allow public client flows**: No
  - **Enable the following mobile and desktop flows**: No
- [ ] Click **Save**

### 4. Configure Token Configuration (Optional)

- [ ] In app registration, go to **Token configuration**
- [ ] Click **Add optional claim**
- [ ] Token type: **ID**
- [ ] Add claims:
  - `email`
  - `family_name`
  - `given_name`
  - `preferred_username`
- [ ] Click **Add**
- [ ] If prompted, check "Turn on the Microsoft Graph email permission" and click **Add**

---

## API Permissions & Consent

### 1. Add Microsoft Graph Delegated Permissions

- [ ] In app registration, go to **API permissions**
- [ ] Click **Add a permission** > **Microsoft Graph** > **Delegated permissions**
- [ ] Search and add each permission:

#### Identity & Directory
- [ ] `User.Read` - Sign in and read user profile (Default)
- [ ] `User.Read.All` - Read all users' full profiles
- [ ] `User.ReadWrite.All` - Read and write all users' full profiles
- [ ] `Directory.Read.All` - Read directory data
- [ ] `Directory.ReadWrite.All` - Read and write directory data
- [ ] `Group.Read.All` - Read all groups
- [ ] `Group.ReadWrite.All` - Read and write all groups
- [ ] `Application.Read.All` - Read all applications
- [ ] `Application.ReadWrite.All` - Read and write all applications

#### Audit & Security
- [ ] `AuditLog.Read.All` - Read audit log data
- [ ] `SecurityEvents.Read.All` - Read your organization's security events
- [ ] `SecurityActions.Read.All` - Read your organization's security actions
- [ ] `IdentityRiskEvent.Read.All` - Read identity risk event information
- [ ] `IdentityRiskyUser.Read.All` - Read identity risk information

#### Device Management
- [ ] `DeviceManagementManagedDevices.Read.All` - Read Microsoft Intune devices
- [ ] `DeviceManagementManagedDevices.ReadWrite.All` - Read and write Microsoft Intune devices
- [ ] `DeviceManagementConfiguration.Read.All` - Read Microsoft Intune device configuration

#### SharePoint & Sites
- [ ] `Sites.Read.All` - Read items in all site collections
- [ ] `Sites.ReadWrite.All` - Edit or delete items in all site collections

#### Reports & Usage
- [ ] `Reports.Read.All` - Read usage reports

#### Token Management
- [ ] `offline_access` - Maintain access to data you have given it access to
- [ ] `openid` - Sign users in
- [ ] `profile` - View users' basic profile
- [ ] `email` - View users' email address

### 2. Add Azure Service Management Permission

- [ ] Click **Add a permission** > **Azure Service Management** > **Delegated permissions**
- [ ] Add:
  - [ ] `user_impersonation` - Access Azure Service Management as organization users

### 3. Grant Admin Consent

**⚠️ CRITICAL STEP**

- [ ] Click **Grant admin consent for {Your Tenant Name}**
- [ ] Review the permissions list in the popup
- [ ] Click **Yes** to confirm
- [ ] Verify all permissions now show **"Granted for {Tenant}"** in the Status column
- [ ] If any permission shows "Not granted", contact your Global Administrator

**Verification**:
```
Status column should show green checkmark with "Granted for {Tenant}" for ALL permissions
```

**Wait Time**: After granting consent, wait **10 minutes** before proceeding to allow permission propagation.

### 4. Document Permissions

- [ ] Take screenshot of API permissions page showing all granted permissions
- [ ] Save to documentation folder for audit trail
- [ ] Document date and administrator who granted consent

---

## Azure Role Assignments

### 1. Identify Service Principal

After creating app registration, a service principal is automatically created.

- [ ] Navigate to **Microsoft Entra ID** > **Enterprise applications**
- [ ] Search for your app name: `Pulse360-Production`
- [ ] Click on the application
- [ ] Copy the **Object ID** (this is the service principal ID)

### 2. Assign Subscription Roles

- [ ] Navigate to **Subscriptions** > Select your production subscription
- [ ] Click **Access control (IAM)**
- [ ] Click **Add** > **Add role assignment**

**Assign Reader Role**:
- [ ] Role: **Reader**
- [ ] Assign access to: **User, group, or service principal**
- [ ] Select members: Search for `Pulse360-Production`
- [ ] Click **Review + assign**

**Assign Security Reader Role** (for Defender for Cloud):
- [ ] Repeat above steps with **Security Reader** role

### 3. Assign Resource-Specific Roles (if needed)

For Log Analytics workspaces:
- [ ] Navigate to each Log Analytics workspace
- [ ] Access control (IAM) > Add role assignment
- [ ] Role: **Log Analytics Reader**
- [ ] Member: `Pulse360-Production`

For Application Insights:
- [ ] Navigate to each Application Insights resource
- [ ] Access control (IAM) > Add role assignment
- [ ] Role: **Monitoring Reader**
- [ ] Member: `Pulse360-Production`

### 4. Verify Role Assignments

```bash
# Use Azure CLI to verify (optional)
az role assignment list --assignee <service-principal-object-id> --all --output table
```

Expected roles:
- Reader (Subscription scope)
- Security Reader (Subscription scope)
- Log Analytics Reader (Workspace scope)
- Monitoring Reader (Application Insights scope)

---

## Power Platform Configuration

### 1. Assign Power Platform Administrator Role

- [ ] Navigate to [Microsoft 365 Admin Center](https://admin.microsoft.com)
- [ ] Go to **Roles** > **Role assignments**
- [ ] Search for **Power Platform Administrator**
- [ ] Click **Assigned** tab
- [ ] Click **Assign admins**
- [ ] Search for and add users who will use Pulse 360
- [ ] Click **Save**

**Alternative**: Assign **Dynamics 365 Administrator** role if Power Platform Administrator is too broad.

### 2. Create Application User in Each Environment

For each Dataverse environment where Pulse 360 will be used:

- [ ] Navigate to [Power Platform Admin Center](https://admin.powerplatform.microsoft.com)
- [ ] Select **Environments**
- [ ] Click on environment name
- [ ] Click **Settings**
- [ ] Expand **Users + permissions**
- [ ] Click **Application users**
- [ ] Click **+ New app user**
- [ ] Click **+ Add an app**
- [ ] Search for your app by Application ID: `<AZURE_CLIENT_ID>`
- [ ] Select the app and click **Add**
- [ ] Select **Business unit**: (Choose appropriate business unit, often root)
- [ ] Click **+ Add security role**
- [ ] Select **System Administrator** (or create custom role with minimum required permissions)
- [ ] Click **Save**

**Repeat for each environment**: Production, Staging, Development (if applicable)

### 3. Test Power Platform Access

- [ ] Use test script to verify access:

```bash
# Run from project directory
node whoami-powerplatform.js
```

Expected output: List of environments

If error occurs, see [TROUBLESHOOTING.md](./TROUBLESHOOTING.md#power-platform-issues)

---

## Security & Secrets Management

### 1. Create Azure Key Vault

**⚠️ CRITICAL FOR PRODUCTION**

- [ ] Navigate to [Azure Portal](https://portal.azure.com)
- [ ] Click **Create a resource** > Search for **Key Vault**
- [ ] Click **Create**
- [ ] Configure:
  - **Resource group**: Select or create production resource group
  - **Key vault name**: `pulse360-prod-kv-<region>` (must be globally unique)
  - **Region**: Same as your app hosting
  - **Pricing tier**: Standard
- [ ] Click **Review + create** > **Create**

### 2. Configure Key Vault Access Policy

- [ ] Open your Key Vault
- [ ] Go to **Access policies**
- [ ] Click **Create**
- [ ] Select permissions:
  - **Secret permissions**: Get, List
- [ ] Click **Next**
- [ ] Select principal: Search for `Pulse360-Production` (your service principal)
- [ ] Click **Next** > **Next** > **Create**

### 3. Add Secrets to Key Vault

**Add each secret**:

- [ ] In Key Vault, go to **Secrets**
- [ ] Click **+ Generate/Import**

**Add these secrets**:

1. **NEXTAUTH-SECRET**:
   - Name: `NEXTAUTH-SECRET`
   - Value: Generate with `openssl rand -base64 32`
   - Click **Create**

2. **AZURE-CLIENT-SECRET**:
   - Name: `AZURE-CLIENT-SECRET`
   - Value: The client secret you saved earlier
   - Click **Create**

**⚠️ Security Checklist**:
- [ ] Verify "Enabled" is set to Yes for all secrets
- [ ] Set expiration date matching client secret expiration
- [ ] Document secret names and creation date
- [ ] **DO NOT** store secrets in:
  - [ ] Source code
  - [ ] Config files committed to Git
  - [ ] Documentation
  - [ ] Email or chat
  - [ ] Screenshots

### 4. Configure Application to Use Key Vault

**For Azure App Service**:
- [ ] In App Service, go to **Configuration** > **Application settings**
- [ ] Add new settings using Key Vault references:

```
NEXTAUTH_SECRET=@Microsoft.KeyVault(SecretUri=https://pulse360-prod-kv.vault.azure.net/secrets/NEXTAUTH-SECRET/)
AZURE_CLIENT_SECRET=@Microsoft.KeyVault(SecretUri=https://pulse360-prod-kv.vault.azure.net/secrets/AZURE-CLIENT-SECRET/)
```

**For Kubernetes**:
Use Azure Key Vault Provider for Secrets Store CSI Driver

**For Docker**:
Mount secrets as files or use environment variables with Key Vault integration

### 5. Enable Key Vault Logging

- [ ] In Key Vault, go to **Diagnostic settings**
- [ ] Click **Add diagnostic setting**
- [ ] Name: `audit-logs`
- [ ] Select:
  - [ ] **AuditEvent**
- [ ] Send to:
  - [ ] Log Analytics workspace (recommended)
- [ ] Click **Save**

### 6. Rotate Secrets (Schedule)

**Set up secret rotation schedule**:

- [ ] Document client secret expiration date: `__________`
- [ ] Set calendar reminder 30 days before expiration
- [ ] Document rotation procedure in runbook
- [ ] Assign secret rotation responsibility to: `__________`

**Rotation Process** (brief):
1. Create new client secret in Azure Portal
2. Add new secret to Key Vault
3. Update App Service configuration to use new secret
4. Test application functionality
5. Delete old client secret after 7 days

---

## Application Configuration

### 1. Environment Variables (Production)

Configure these in your hosting environment (Azure App Service, Kubernetes, etc.):

**Required**:
```bash
# Application URL
NEXTAUTH_URL=https://your-production-domain.com

# Authentication (from Key Vault)
NEXTAUTH_SECRET=@Microsoft.KeyVault(...)
AZURE_CLIENT_ID=<your-client-id>
AZURE_CLIENT_SECRET=@Microsoft.KeyVault(...)
AZURE_TENANT_ID=<your-tenant-id>

# Alternative names (NextAuth v5 compatibility)
AUTH_SECRET=@Microsoft.KeyVault(...)
AUTH_AZURE_AD_CLIENT_ID=<your-client-id>
AUTH_AZURE_AD_CLIENT_SECRET=@Microsoft.KeyVault(...)
AUTH_AZURE_AD_TENANT_ID=<your-tenant-id>

# Azure (optional)
AZURE_SUBSCRIPTION_ID=<your-subscription-id>
```

**Optional** (based on your needs):
```bash
# Defender for Cloud
DEFENDER_MANAGEMENT_SCOPE=https://management.azure.com/.default

# Custom API endpoints (if using private endpoints)
GRAPH_API_ENDPOINT=https://graph.microsoft.com
POWER_PLATFORM_API_ENDPOINT=https://api.powerplatform.com
```

**Do NOT set in production**:
```bash
# These should NEVER be in production
NODE_TLS_REJECT_UNAUTHORIZED=0  # NEVER in production
AUTH_DISABLED=1                  # NEVER in production
```

### 2. Verify Configuration

- [ ] All required environment variables are set
- [ ] Secrets are sourced from Key Vault
- [ ] `NEXTAUTH_URL` matches actual production domain
- [ ] No secrets are in plain text in configuration
- [ ] No development/debugging flags enabled

### 3. Build Configuration

**Update `next.config.ts` for production**:

```typescript
const nextConfig = {
  // ... existing config
  
  // Ensure proper CSP headers
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'X-Frame-Options',
            value: 'DENY'
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff'
          },
          {
            key: 'Referrer-Policy',
            value: 'strict-origin-when-cross-origin'
          }
        ]
      }
    ]
  },
  
  // Production optimizations
  reactStrictMode: true,
  swcMinify: true,
  
  // Disable source maps in production (optional, for security)
  productionBrowserSourceMaps: false,
}
```

---

## Testing & Validation

### 1. Local Testing

**Before deploying**, test locally with production configuration:

- [ ] Set `.env.local` to use production app registration (with development redirect URI)
- [ ] Run: `npm run build`
- [ ] Verify build completes without errors
- [ ] Run: `npm start`
- [ ] Test all critical paths:
  - [ ] User can sign in
  - [ ] Users page loads
  - [ ] Groups page loads
  - [ ] Power Platform environments load
  - [ ] Azure resources load
  - [ ] Security alerts load
  - [ ] Reports generate successfully

### 2. Staging Environment Testing

**Deploy to staging environment first**:

- [ ] Deploy application to staging
- [ ] Configure staging environment variables
- [ ] Use staging subdomain: `https://staging.your-domain.com`
- [ ] Add staging redirect URI to app registration

**Staging Test Checklist**:
- [ ] **Authentication**: 5 users can sign in successfully
- [ ] **Users Management**: Can list, view details, create test user
- [ ] **Groups**: Can list groups and view members
- [ ] **Applications**: Can list app registrations and service principals
- [ ] **Audit Logs**: Can view directory audits and sign-in logs
- [ ] **Security**: Secure Score, Risky Users, Alerts load
- [ ] **Azure Resources**: Resource Graph queries return results
- [ ] **Device Management**: Intune devices load
- [ ] **Power Platform**: Environments load, can view teams/solutions
- [ ] **SharePoint**: Sites load with correct data
- [ ] **Reports**: Usage reports generate correctly
- [ ] **Performance**: All pages load within 3 seconds
- [ ] **Mobile**: Test on mobile devices (responsive design)
- [ ] **Browser Compatibility**: Test on Chrome, Edge, Firefox, Safari

### 3. Load Testing (Optional but Recommended)

- [ ] Use tool like Apache Bench or Artillery
- [ ] Simulate 50 concurrent users
- [ ] Verify application remains stable
- [ ] Check Azure monitoring for throttling/errors

```bash
# Example load test
ab -n 1000 -c 10 https://staging.your-domain.com/users
```

### 4. Security Testing

- [ ] Run security scan using:
  - [ ] npm audit
  - [ ] OWASP ZAP (optional)
  - [ ] Azure Security Center recommendations
- [ ] Verify no secrets in:
  - [ ] Browser DevTools Network tab
  - [ ] Client-side JavaScript
  - [ ] Error messages
- [ ] Test authentication:
  - [ ] Cannot access pages without login
  - [ ] Session expires appropriately
  - [ ] Logout works correctly

---

## Monitoring & Logging

### 1. Application Insights Setup

- [ ] Create Application Insights resource in Azure Portal
- [ ] Configure connection string in App Service:
  ```
  APPLICATIONINSIGHTS_CONNECTION_STRING=InstrumentationKey=...;IngestionEndpoint=...
  ```
- [ ] Enable Application Insights in code (if not already):
  ```javascript
  // next.config.ts
  module.exports = {
    // ... other config
    experimental: {
      instrumentationHook: true
    }
  }
  ```

### 2. Configure Alerts

**Create alerts for**:

- [ ] **Application Errors**
  - Threshold: > 10 errors in 5 minutes
  - Action: Email admin team

- [ ] **Failed Authentication**
  - Threshold: > 20 failed logins in 5 minutes
  - Action: Email security team

- [ ] **API Response Time**
  - Threshold: Average > 5 seconds
  - Action: Email dev team

- [ ] **Availability**
  - Threshold: < 99% uptime
  - Action: Email on-call engineer

### 3. Log Analytics Workspace

- [ ] Create Log Analytics workspace (if not exists)
- [ ] Link Application Insights to workspace
- [ ] Configure retention: 90 days (recommended)
- [ ] Set up custom queries for:
  - User login patterns
  - API usage statistics
  - Error trends

### 4. Azure Monitor Dashboard

- [ ] Create Azure Monitor dashboard
- [ ] Add tiles for:
  - Application availability
  - Request count
  - Failed requests
  - Average response time
  - User count
- [ ] Share dashboard with operations team

---

## Production Deployment

### 1. Pre-Deployment Checklist

**Verify ALL items are complete**:

- [ ] App registration configured
- [ ] All API permissions granted
- [ ] Azure roles assigned
- [ ] Power Platform access configured
- [ ] Secrets in Key Vault
- [ ] Staging testing completed successfully
- [ ] Monitoring and alerts configured
- [ ] Documentation updated
- [ ] Team trained on application
- [ ] Rollback plan documented
- [ ] Incident response plan in place

### 2. Deployment Window

**Schedule deployment**:
- [ ] Date: `_______________`
- [ ] Time: `_______________` (recommend off-hours)
- [ ] Duration: `_______________` (estimate 2-4 hours)
- [ ] On-call engineer: `_______________`
- [ ] Rollback engineer: `_______________`

### 3. Deployment Steps

**For Azure App Service**:

1. [ ] Build application:
   ```bash
   npm run build
   ```

2. [ ] Create deployment package:
   ```bash
   zip -r deploy.zip .next public package.json package-lock.json
   ```

3. [ ] Deploy to Azure App Service:
   ```bash
   az webapp deploy --resource-group <rg-name> --name <app-name> --src-path deploy.zip --type zip
   ```

4. [ ] Configure environment variables in App Service:
   - [ ] Application settings (see [Application Configuration](#application-configuration))

5. [ ] Restart App Service:
   ```bash
   az webapp restart --resource-group <rg-name> --name <app-name>
   ```

**For Docker/Kubernetes**:

1. [ ] Build Docker image:
   ```bash
   docker build -t pulse360:latest .
   ```

2. [ ] Push to container registry:
   ```bash
   docker push yourregistry.azurecr.io/pulse360:latest
   ```

3. [ ] Deploy to Kubernetes:
   ```bash
   kubectl apply -f k8s/deployment.yaml
   kubectl rollout status deployment/pulse360
   ```

### 4. Post-Deployment Smoke Test

**Immediately after deployment**:

- [ ] Application URL is accessible
- [ ] Login page loads
- [ ] Can authenticate successfully
- [ ] Home page displays without errors
- [ ] Critical pages load (users, groups, environments)
- [ ] Check Application Insights for errors
- [ ] Monitor for 15 minutes

---

## Post-Deployment Verification

### 1. Functional Testing (Production)

**Test with real production user accounts**:

- [ ] Admin user can sign in
- [ ] Standard user can sign in
- [ ] Users page displays production data
- [ ] Groups page displays correct groups
- [ ] Power Platform environments are listed
- [ ] Azure resources query works
- [ ] Reports generate with production data
- [ ] Audit logs display correctly

### 2. Performance Verification

- [ ] Measure page load times:
  - [ ] Home page: `_______` seconds (target: < 2s)
  - [ ] Users page: `_______` seconds (target: < 3s)
  - [ ] Power Platform: `_______` seconds (target: < 5s)
- [ ] Check Application Insights for:
  - [ ] Average response time: `_______` ms (target: < 500ms)
  - [ ] Failed requests: `_______` (target: 0)

### 3. Security Verification

- [ ] No secrets visible in browser Network tab
- [ ] HTTPS enforced (no HTTP)
- [ ] Security headers present (X-Frame-Options, etc.)
- [ ] Authentication required for all pages
- [ ] Session timeout works correctly

### 4. Monitoring Verification

- [ ] Application Insights receiving telemetry
- [ ] Logs appearing in Log Analytics workspace
- [ ] Alerts configured and active
- [ ] Dashboard showing real-time metrics

---

## Rollback Plan

**If deployment fails or critical issues arise**:

### Immediate Rollback Steps

1. [ ] **Stop new traffic** to production environment
2. [ ] **Revert to previous deployment**:
   
   **Azure App Service**:
   ```bash
   # Swap back to previous slot
   az webapp deployment slot swap --resource-group <rg> --name <app> --slot staging --target-slot production
   ```
   
   **Kubernetes**:
   ```bash
   kubectl rollout undo deployment/pulse360
   ```

3. [ ] **Verify rollback successful**:
   - [ ] Application accessible
   - [ ] Users can sign in
   - [ ] Critical functionality works

4. [ ] **Document the incident**:
   - [ ] What went wrong?
   - [ ] What was the impact?
   - [ ] How long was the outage?
   - [ ] What steps were taken?

### Rollback Decision Criteria

**Rollback immediately if**:
- Users cannot sign in
- Critical data not loading (users, groups, environments)
- Widespread errors (> 10% of requests failing)
- Security issue discovered
- Data corruption detected

**Investigate first if**:
- Minor cosmetic issues
- Single user reporting issue
- Performance slightly degraded but acceptable
- Non-critical feature broken

---

## Communication Plan

### Pre-Deployment Communication

**1 week before**:
- [ ] Email to all users announcing deployment
- [ ] Include:
  - Deployment date and time
  - Expected downtime (if any)
  - New features/changes
  - Support contact information

**1 day before**:
- [ ] Reminder email
- [ ] Final testing window for stakeholders

### During Deployment

- [ ] Status page updated (if available)
- [ ] Real-time updates in team chat
- [ ] Incident bridge open (if needed)

### Post-Deployment Communication

**Within 1 hour**:
- [ ] Email confirming successful deployment
- [ ] Known issues (if any)
- [ ] How to report problems

**Within 24 hours**:
- [ ] Summary report:
  - Deployment timeline
  - Any issues encountered
  - Resolution steps
  - Lessons learned

---

## Documentation Updates

**After successful deployment**:

- [ ] Update README.md with production URL
- [ ] Update environment variable documentation
- [ ] Document any production-specific configurations
- [ ] Create runbook for common operations
- [ ] Update architecture diagrams (if changed)
- [ ] Document troubleshooting steps for production issues
- [ ] Create user guide or training materials

---

## Ongoing Maintenance

### Weekly Tasks

- [ ] Review Application Insights for errors
- [ ] Check performance metrics
- [ ] Review security alerts
- [ ] Verify backup jobs (if applicable)

### Monthly Tasks

- [ ] Review and rotate logs
- [ ] Update dependencies:
  ```bash
  npm audit
  npm outdated
  npm update
  ```
- [ ] Review Azure costs
- [ ] Test disaster recovery procedures
- [ ] Review access logs for suspicious activity

### Quarterly Tasks

- [ ] Review and update API permissions (if needed)
- [ ] Review Azure role assignments
- [ ] Update documentation
- [ ] Conduct security review
- [ ] Load test production environment
- [ ] Review and update incident response plan

### Annual Tasks

- [ ] Rotate client secret (before expiration!)
- [ ] Review architecture for improvements
- [ ] Major version upgrades (Next.js, React, etc.)
- [ ] Comprehensive security audit
- [ ] Disaster recovery drill

---

## Appendix: Contact Information

### Key Personnel

| Role | Name | Email | Phone | Backup |
|------|------|-------|-------|--------|
| Azure Administrator | _______ | _______ | _______ | _______ |
| Application Owner | _______ | _______ | _______ | _______ |
| Security Lead | _______ | _______ | _______ | _______ |
| DevOps Engineer | _______ | _______ | _______ | _______ |
| On-Call Engineer | _______ | _______ | _______ | _______ |

### Support Contacts

- **IT Help Desk**: helpdesk@company.com | +1-555-0100
- **Security Team**: security@company.com | +1-555-0200
- **DevOps Team**: devops@company.com | +1-555-0300

### External Contacts

- **Microsoft Support**: [Azure Support Portal](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade)
- **Microsoft Premier Support**: +1-800-MICROSOFT (if applicable)

---

## Sign-Off

**Deployment approved by**:

| Role | Name | Signature | Date |
|------|------|-----------|------|
| IT Manager | _______ | _______ | _______ |
| Security Manager | _______ | _______ | _______ |
| Application Owner | _______ | _______ | _______ |

**Deployment executed by**:

| Role | Name | Signature | Date |
|------|------|-----------|------|
| DevOps Engineer | _______ | _______ | _______ |

---

**Document Version:** 1.0  
**Last Updated:** November 1, 2025  
**Next Review Date:** February 1, 2026

For detailed API configuration, see [API-DOCUMENTATION.md](./API-DOCUMENTATION.md).  
For troubleshooting assistance, see [TROUBLESHOOTING.md](./TROUBLESHOOTING.md).
