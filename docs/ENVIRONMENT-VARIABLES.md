# Environment Variables Configuration

This file documents all environment variables used by Pulse 360°.

## Required Variables (Authentication)

```bash
# NextAuth Configuration
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=your-random-secret-key-here

# Azure AD Configuration
AZURE_TENANT_ID=your-tenant-id
AZURE_CLIENT_ID=your-client-id
AZURE_CLIENT_SECRET=your-client-secret
```

## Optional Variables (Features)

### Azure Subscription (for Azure Resource Graph)
```bash
AZURE_SUBSCRIPTION_ID=your-subscription-id
```

### Microsoft Defender for Endpoint
```bash
DEFENDER_ATP_TENANT_ID=your-tenant-id
DEFENDER_ATP_CLIENT_ID=your-defender-app-client-id
DEFENDER_ATP_CLIENT_SECRET=your-defender-app-secret
```

### ServiceNow Integration
```bash
# ServiceNow instance name (e.g., dev12345, mycompany)
# URL will be constructed as: https://{instance}.service-now.com
SERVICENOW_INSTANCE=dev12345

# ServiceNow API credentials
SERVICENOW_USERNAME=your-servicenow-username
SERVICENOW_PASSWORD=your-servicenow-password
```

## Usage Notes

### ServiceNow Configuration
When `SERVICENOW_INSTANCE`, `SERVICENOW_USERNAME`, and `SERVICENOW_PASSWORD` are configured:
- The instance field will be pre-filled and disabled in the Create Incident form
- Users only need to enter incident details
- Credentials are kept secure on the server

If these variables are NOT configured:
- Users will be prompted to enter ServiceNow connection details each time
- Connection details are not stored (for security)

### Environment Variable Priority
For ServiceNow, the priority order is:
1. Environment variables (`.env.local`)
2. User-provided values (form input)

This allows flexibility while maintaining security best practices.

## Example .env.local File

```bash
# Authentication (Required)
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=your-secret-here
AZURE_TENANT_ID=12345678-1234-1234-1234-123456789012
AZURE_CLIENT_ID=12345678-1234-1234-1234-123456789012
AZURE_CLIENT_SECRET=your~secret~here

# Azure Resources (Optional)
AZURE_SUBSCRIPTION_ID=12345678-1234-1234-1234-123456789012

# Defender ATP (Optional)
DEFENDER_ATP_TENANT_ID=12345678-1234-1234-1234-123456789012
DEFENDER_ATP_CLIENT_ID=12345678-1234-1234-1234-123456789012
DEFENDER_ATP_CLIENT_SECRET=your~defender~secret

# ServiceNow (Optional)
SERVICENOW_INSTANCE=dev12345
SERVICENOW_USERNAME=admin
SERVICENOW_PASSWORD=your-password-here
```

## Security Best Practices

1. ✅ **Never commit** `.env.local` to version control
2. ✅ **Use strong, unique secrets** for `NEXTAUTH_SECRET`
3. ✅ **Rotate credentials regularly** for ServiceNow and Azure AD
4. ✅ **Use separate credentials** for development and production
5. ✅ **Limit Azure AD app permissions** to only what's needed
6. ✅ **Store production secrets** in Azure Key Vault or similar

## Troubleshooting

### ServiceNow Incidents Not Creating
- Verify `SERVICENOW_INSTANCE` is just the instance name (not full URL)
- Confirm credentials have permission to create incidents
- Check ServiceNow instance is accessible from your network

### Authentication Errors
- Ensure `NEXTAUTH_URL` matches your actual URL (including port)
- Verify `AZURE_TENANT_ID` matches your Azure AD tenant
- Check client secret hasn't expired in Azure AD

### API Permission Errors
- Review app registration in Azure AD
- Ensure admin consent has been granted for required permissions
- Check token scopes in browser developer tools
