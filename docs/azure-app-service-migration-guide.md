# Azure App Service Migration Guide - Solution Analysis
**Target:** Move from Vercel to Azure App Service  
**Status:** Current implementation is Azure-ready (optional MSI enhancement available)

## Current Implementation (Works on Both Vercel and Azure)

The solution analysis route at `src/app/api/power-platform/solutions/analyze/route.js` uses:
- Delegated tokens for Dataverse access
- API key for Azure OpenAI access
- Pure Node.js (no platform-specific dependencies)

✅ **Zero code changes needed** to move from Vercel to Azure App Service!

## Optional Enhancement: Managed Identity for Azure OpenAI

Once on Azure App Service, you can eliminate the `AI_API_KEY` by using **Managed Identity**:

### Benefits of MSI for Azure OpenAI
- 🔒 **No secrets in .env**: Key stored securely in Azure
- 🔄 **Auto-rotation**: No manual key rotation needed
- 📊 **Better auditing**: All API calls logged with MSI identity
- 🛡️ **Private endpoints**: Keep traffic within Azure backbone

### Setup Steps (5 minutes)

#### 1. Enable Managed Identity on App Service
```bash
# Enable system-assigned managed identity
az webapp identity assign \
  --name <your-app-service-name> \
  --resource-group <your-rg>

# Get the principal ID
PRINCIPAL_ID=$(az webapp identity show \
  --name <your-app-service-name> \
  --resource-group <your-rg> \
  --query principalId -o tsv)

echo "App Service Principal ID: $PRINCIPAL_ID"
```

#### 2. Grant MSI Access to Azure OpenAI
```bash
# Get your Azure OpenAI resource ID
OPENAI_RESOURCE_ID=$(az cognitiveservices account show \
  --name messagecenterai \
  --resource-group <your-rg> \
  --query id -o tsv)

# Grant "Cognitive Services OpenAI User" role to MSI
az role assignment create \
  --assignee $PRINCIPAL_ID \
  --role "Cognitive Services OpenAI User" \
  --scope $OPENAI_RESOURCE_ID
```

#### 3. Update Environment Variables
**Remove:**
```bash
# AI_API_KEY=<key>  # No longer needed with MSI
```

**Add:**
```bash
USE_MANAGED_IDENTITY=true  # Enable MSI for Azure OpenAI
```

#### 4. Update Solution Analysis Route (5-line change)

```javascript
// In src/app/api/power-platform/solutions/analyze/route.js

// BEFORE (line ~185):
const aiApiKey = process.env.AI_API_KEY || process.env.GITHUB_TOKEN

if (!aiApiKey) {
  return NextResponse.json({
    success: false,
    error: 'AI not configured. Please set AI_API_KEY or GITHUB_TOKEN environment variable.'
  }, { status: 500 })
}

// AFTER (add this import at top):
import { DefaultAzureCredential } from '@azure/identity'

// AFTER (replace lines 185-193):
let authHeader
if (process.env.USE_MANAGED_IDENTITY === 'true') {
  // Use Managed Identity (Azure App Service only)
  const credential = new DefaultAzureCredential()
  const token = await credential.getToken('https://cognitiveservices.azure.com/.default')
  authHeader = `Bearer ${token.token}`
  console.log('[Solution Analyze] Using Managed Identity for Azure OpenAI')
} else {
  // Use API key (Vercel or Azure with key auth)
  const aiApiKey = process.env.AI_API_KEY || process.env.GITHUB_TOKEN
  if (!aiApiKey) {
    return NextResponse.json({
      success: false,
      error: 'AI not configured. Please set AI_API_KEY or GITHUB_TOKEN environment variable.'
    }, { status: 500 })
  }
  authHeader = `Bearer ${aiApiKey}`
  console.log('[Solution Analyze] Using API key for Azure OpenAI')
}

// Then use authHeader instead of aiApiKey:
const aiResponse = await fetch(`${aiEndpoint}/chat/completions`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': authHeader  // Changed from: `Bearer ${aiApiKey}`
  },
  body: JSON.stringify({ ... })
})
```

## Deployment Comparison

### Vercel (Current)
```
✅ Simple deployment (git push)
✅ Automatic HTTPS
✅ Global CDN
✅ Free tier available
⚠️ Requires API key for Azure OpenAI
⚠️ No VNet integration
⚠️ Cold starts on free tier
```

### Azure App Service (Recommended for Enterprise)
```
✅ VNet integration (private networking)
✅ Managed Identity (no secrets)
✅ Private endpoints (Azure OpenAI never leaves Azure)
✅ Always-on (no cold starts)
✅ Better monitoring (App Insights native)
✅ Compliance (data residency, certifications)
⚠️ Requires Azure setup (~15 minutes)
```

## Cost Analysis: Azure App Service vs. Azure Functions

### Option 1: Azure App Service (✅ Recommended)
```
Basic (B1): $13/month
- 1 core, 1.75 GB RAM
- Good for dev/test

Standard (S1): $73/month
- 1 core, 1.75 GB RAM
- Auto-scaling, slots, VNet

Premium (P1v3): $117/month
- 2 cores, 8 GB RAM
- Private endpoints, better performance
```

**Includes:**
- Unlimited HTTP requests
- Next.js hosting
- All API routes
- Always-on instances

**Additional costs:**
- Azure OpenAI: ~$0.03 per solution analysis
- Application Insights: Free tier (5 GB/month)

**Total: $73-117/month + $0.03/analysis**

### Option 2: Durable Functions (❌ More Complex)
```
Function App (Consumption): $0.20/million executions
- Cold starts
- 5-minute max duration (need Premium for longer)

Function App (Premium EP1): $169/month
- Always-on
- No cold start
- VNet integration

Storage Account: $10/month
- Blob storage for orchestration
- Queue storage for state

Azure OpenAI: $0.03/analysis
```

**Total: $179/month + $0.03/analysis** (54% more expensive than App Service!)

**Drawbacks:**
- More complex architecture (3 services instead of 1)
- Requires polling or webhooks
- Harder to debug
- More secrets to manage

## Migration Checklist

### Phase 1: Move to Azure App Service (No Code Changes)
1. ✅ Create App Service (Node 18+ runtime)
   ```bash
   az webapp create \
     --name pulse360-portal \
     --resource-group <your-rg> \
     --plan <app-service-plan> \
     --runtime "NODE|18-lts"
   ```

2. ✅ Configure environment variables
   ```bash
   az webapp config appsettings set \
     --name pulse360-portal \
     --resource-group <your-rg> \
     --settings \
       AZURE_CLIENT_ID=<value> \
       AZURE_CLIENT_SECRET=<value> \
       AZURE_TENANT_ID=<value> \
       AI_API_KEY=<value> \
       AI_ENDPOINT=<value> \
       NEXTAUTH_URL=<your-app-service-url> \
       AUTH_SECRET=<value>
   ```

3. ✅ Enable Application Insights (already configured)
   ```bash
   az webapp config appsettings set \
     --name pulse360-portal \
     --resource-group <your-rg> \
     --settings \
       APPLICATIONINSIGHTS_CONNECTION_STRING=<your-connection-string>
   ```

4. ✅ Deploy code
   - **Option A:** GitHub Actions (recommended)
   - **Option B:** Azure DevOps
   - **Option C:** `az webapp deployment source config-local-git`

5. ✅ Test solution analysis feature

**Expected time: 30 minutes**  
**Downtime: 0 minutes** (Vercel continues running until cutover)

### Phase 2: Enhanced Security with Managed Identity (Optional)
1. ✅ Enable MSI on App Service
2. ✅ Grant MSI access to Azure OpenAI
3. ✅ Update solution analysis route (5-line change)
4. ✅ Remove `AI_API_KEY` from environment variables
5. ✅ Test solution analysis with MSI

**Expected time: 15 minutes**  
**Downtime: 0 minutes** (deploy with feature flag, test, then enable)

### Phase 3: Advanced Azure Features (Future)
1. ⏳ VNet integration (lock down to corporate network)
2. ⏳ Private endpoints for Azure OpenAI (no public internet)
3. ⏳ Azure Front Door (global CDN + WAF)
4. ⏳ Multiple deployment slots (blue-green deployments)

## Architecture Comparison

### Current (Vercel)
```
Internet → Vercel Edge → Next.js API Routes → Azure OpenAI (public)
                                            → Dataverse (public)
```

### Enhanced (Azure App Service)
```
Internet → Azure Front Door → App Service (VNet) → Azure OpenAI (private endpoint)
                                                  → Dataverse (public or private link)
```

## Key Takeaway

**The direct AI analysis pattern is the BEST choice for Azure App Service:**

✅ **Simpler** than Durable Functions (1 service vs 3)  
✅ **Cheaper** ($73/month vs $179/month)  
✅ **Faster** (no cold starts, no orchestration overhead)  
✅ **More secure** (VNet + MSI + private endpoints available)  
✅ **Easier to monitor** (single service, native App Insights)  
✅ **No code changes** needed to migrate from Vercel  

**Bonus:** You can still use the Function App for other scenarios (batch processing, scheduled jobs, etc.) but solution analysis doesn't need it.

## Recommendation

1. **Now (Vercel):** Keep using current implementation with API key
2. **Short term (1-2 months):** Migrate to Azure App Service (30 min setup, no code changes)
3. **Medium term (3-6 months):** Enable Managed Identity (5-line code change)
4. **Long term (6-12 months):** Add VNet + private endpoints (enterprise security)

The architecture you have is already optimal for Azure! 🎉
