# Azure Bicep Deployment - Azure CLI to SDK Migration

## Summary

Successfully migrated the Azure Bicep deployment feature from **Azure CLI dependency** to **Azure SDK for JavaScript** (`@azure/arm-resources`). This makes the deployment feature **cloud-native** and enables it to work in both **local development** and **production environments** (Azure App Service, containers, etc.).

## Changes Made

### 1. Backend API Route (`src/app/api/azure/bicep/deploy/route.js`)

#### Before (Azure CLI approach):
- Used Node.js `spawn()` to execute `az deployment sub create` commands
- Required Azure CLI to be installed on the server
- Created temporary files on disk
- Only worked locally where Azure CLI was installed
- **Would NOT work on Azure App Service**

#### After (Azure SDK approach):
- Uses `@azure/arm-resources` SDK with `ResourceManagementClient`
- Authenticates via `ClientSecretCredential` using environment variables
- No temporary files needed - Bicep content passed directly to SDK
- Works in **any environment** with network access to Azure
- **Works on Azure App Service, containers, and local development**

### 2. Authentication Method

#### Before:
```javascript
spawn('az', ['account', 'show'])  // Checked for Azure CLI authentication
```

#### After:
```javascript
const credential = new ClientSecretCredential(
  process.env.AZURE_TENANT_ID,
  process.env.AZURE_CLIENT_ID,
  process.env.AZURE_CLIENT_SECRET
);
const client = new ResourceManagementClient(credential, subscriptionId);
```

### 3. Deployment Execution

#### Before:
```javascript
spawn('az', [
  'deployment', 'sub', 'create',
  '--template-file', mainBicepPath,
  '--parameters', ...
]);
```

#### After:
```javascript
const deploymentOperation = await client.deployments.beginCreateOrUpdateAtSubscriptionScope(
  deploymentName,
  {
    properties: {
      mode: 'Incremental',
      template: mainBicepContent,
      parameters: deploymentParameters
    }
  }
);
const result = await deploymentOperation.pollUntilDone();
```

### 4. Frontend Updates (`src/app/azure/deploy/page.jsx`)

- Updated error messages to reflect SDK-based authentication
- Changed from "Azure CLI not installed" to "Azure credentials not configured"
- Shows missing environment variables in error messages
- Displays tenant ID and subscription ID when connected

## Dependencies Added

```bash
npm install @azure/arm-resources
```

**Note:** `@azure/identity` was already installed in the project.

## Environment Variables Required

The deployment feature now requires these environment variables:

```bash
# Required for authentication
AZURE_TENANT_ID=your-tenant-id
AZURE_CLIENT_ID=your-client-id
AZURE_CLIENT_SECRET=your-client-secret

# Optional - can also be passed in request body
AZURE_SUBSCRIPTION_ID=your-subscription-id
```

These should already be configured for other Azure features in the app.

## API Endpoint Changes

### GET `/api/azure/bicep/deploy`

#### Before Response:
```json
{
  "azureCliInstalled": true,
  "authenticated": true,
  "account": {
    "user": { "name": "..." },
    "tenantId": "...",
    ...
  }
}
```

#### After Response:
```json
{
  "configured": true,
  "authenticated": true,
  "subscriptionId": "...",
  "hasSubscription": true,
  "tenantId": "...",
  "clientId": "...",
  "message": "Ready to deploy"
}
```

### POST `/api/azure/bicep/deploy`

Request body format remains the same, with optional `subscriptionId`:

```json
{
  "files": {
    "main.bicep": "..."
  },
  "deploymentName": "my-deployment",
  "location": "westeurope",
  "parameters": { ... },
  "subscriptionId": "optional-subscription-id"
}
```

## Benefits

1. **Cloud-Native**: Works anywhere Node.js runs with network access
2. **No External Dependencies**: No need for Azure CLI installation
3. **Production-Ready**: Deploys seamlessly to Azure App Service
4. **Better Error Handling**: Structured error messages from Azure SDK
5. **Simpler Deployment**: No file system operations, cleaner code
6. **Container-Friendly**: Works in Docker, Kubernetes, etc.

## Testing

Build completed successfully:
```bash
npm run build
✓ Compiled successfully
✓ All routes built correctly
```

## Migration Path for Other Features

This pattern can be applied to other features that currently use Azure CLI:
1. Replace `spawn('az', ...)` with appropriate Azure SDK client
2. Use `ClientSecretCredential` for authentication
3. Update error handling to check for environment variables
4. Update frontend messages accordingly

## Documentation References

- [Azure SDK for JavaScript - Resource Management](https://learn.microsoft.com/en-us/javascript/api/overview/azure/arm-resources-readme)
- [Deploy Bicep with Azure SDK](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/deploy-rest)
- [@azure/arm-resources NPM Package](https://www.npmjs.com/package/@azure/arm-resources)

## Date

December 2, 2025
