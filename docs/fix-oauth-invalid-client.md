# Fixing OAuth invalid_client Error on Vercel

## Error
```
OAuthCallbackError: OAuth Provider returned an error: invalid_client
```

This error occurs when Azure AD rejects the authentication attempt from your Vercel deployment.

## Root Causes & Solutions

### 1. Missing or Incorrect Client Secret (Most Common)

**Problem**: The `AZURE_CLIENT_SECRET` environment variable in Vercel is missing, incorrect, or expired.

**Solution**:
1. Go to [Azure Portal](https://portal.azure.com)
2. Navigate to: **Azure Active Directory** → **App registrations** → **[Your App]**
3. Click **Certificates & secrets**
4. Check if your client secret has expired
5. If expired or missing, create a new secret:
   - Click **New client secret**
   - Add description: "Vercel Production"
   - Set expiration (recommend: 24 months)
   - **COPY THE VALUE IMMEDIATELY** (you can't see it again!)
6. Go to [Vercel Dashboard](https://vercel.com) → Your Project → Settings → Environment Variables
7. Add/Update: `AZURE_CLIENT_SECRET` = `[your new secret value]`
8. **Important**: Redeploy your app for changes to take effect

### 2. Missing Redirect URI Configuration

**Problem**: Your Vercel domain is not registered as an allowed redirect URI in Azure AD.

**Solution**:
1. Go to [Azure Portal](https://portal.azure.com)
2. Navigate to: **Azure Active Directory** → **App registrations** → **[Your App]**
3. Click **Authentication** in the left menu
4. Under **Platform configurations** → **Web**, add redirect URIs:
   ```
   https://[your-vercel-domain].vercel.app/api/auth/callback/azure-ad
   ```
   
   **Example**:
   ```
   https://portalofportals.vercel.app/api/auth/callback/azure-ad
   ```

5. If you have **preview deployments**, you may need wildcards (not recommended for production):
   ```
   https://portalofportals-*.vercel.app/api/auth/callback/azure-ad
   ```
   
   Or add each preview domain individually.

6. Click **Save**

### 3. Missing Environment Variables in Vercel

**Problem**: Required NextAuth environment variables are not set in Vercel.

**Solution**: Verify all environment variables are set in Vercel:

#### Required Variables:
```bash
AZURE_CLIENT_ID=32c72b33-xxxx-xxxx-xxxx-xxxxxxxxxxxx
AZURE_CLIENT_SECRET=your-secret-value-from-azure
AZURE_TENANT_ID=f33d7d7f-xxxx-xxxx-xxxx-xxxxxxxxxxxx
AUTH_SECRET=generate-random-string-here
AUTH_URL=https://portalofportals.vercel.app
```

#### Generate AUTH_SECRET:
```bash
openssl rand -base64 32
```

Or use NextAuth CLI:
```bash
npx auth secret
```

### 4. Client ID Mismatch

**Problem**: The `AZURE_CLIENT_ID` in Vercel doesn't match the App Registration in Azure AD.

**Solution**:
1. Go to Azure Portal → Azure Active Directory → App registrations → [Your App]
2. Copy the **Application (client) ID** from the Overview page
3. Verify it matches `AZURE_CLIENT_ID` in Vercel environment variables exactly
4. Update if different and redeploy

### 5. Wrong Tenant ID

**Problem**: Using 'common' tenant when app is single-tenant, or using specific tenant ID when app is multi-tenant.

**Solution**:
1. Check your app registration type in Azure Portal:
   - **Single tenant**: Use specific tenant ID (`AZURE_TENANT_ID=f33d7d7f-...`)
   - **Multi-tenant**: Can use 'common' or specific tenant ID
2. Verify `AZURE_TENANT_ID` matches your app's tenant configuration
3. Update Vercel environment variable if needed and redeploy

## Verification Steps

### Step 1: Check Azure AD Configuration
```bash
✓ App registration exists
✓ Client ID is correct
✓ Client secret is valid (not expired)
✓ Redirect URI includes: https://[your-domain].vercel.app/api/auth/callback/azure-ad
✓ Tenant ID matches app registration
```

### Step 2: Check Vercel Environment Variables
Log into Vercel Dashboard → Settings → Environment Variables and verify:
```bash
✓ AZURE_CLIENT_ID (matches Azure App Registration)
✓ AZURE_CLIENT_SECRET (valid, not expired)
✓ AZURE_TENANT_ID (matches Azure tenant)
✓ AUTH_SECRET (random string, 32+ characters)
✓ AUTH_URL (your production domain with https://)
```

### Step 3: Test Authentication Flow
1. Redeploy your Vercel app after making changes
2. Visit your production URL: `https://[your-domain].vercel.app`
3. Click sign in
4. Should redirect to Microsoft login
5. After successful login, should redirect back to your app

## Debugging Commands

### View NextAuth Configuration (Production)
Already logging in your auth.ts file:
```typescript
console.log('[auth] Production configuration:', {
  hasClientId: !!clientId,
  clientIdPrefix: clientId ? clientId.substring(0, 8) + '...' : 'missing',
  hasClientSecret: !!clientSecret,
  hasTenantId: !!tenantId,
  hasSecret: !!resolvedSecret,
  authUrl: process.env.AUTH_URL || process.env.NEXTAUTH_URL || 'not set',
  issuer
})
```

Check Vercel logs to see this output and verify all values are present.

### Test Redirect URI
Open browser and navigate to:
```
https://login.microsoftonline.com/[TENANT_ID]/oauth2/v2.0/authorize?client_id=[CLIENT_ID]&response_type=code&redirect_uri=https://[YOUR_DOMAIN].vercel.app/api/auth/callback/azure-ad&scope=openid profile email
```

Replace:
- `[TENANT_ID]` with your Azure tenant ID
- `[CLIENT_ID]` with your Azure client ID
- `[YOUR_DOMAIN]` with your Vercel domain

**Expected**: Should show Microsoft login page
**Error**: "AADSTS50011" = invalid redirect URI (not registered)

## Common Errors & Meanings

| Error Code | Meaning | Solution |
|------------|---------|----------|
| `invalid_client` | Client authentication failed | Check client secret, client ID, tenant ID |
| `AADSTS50011` | Redirect URI mismatch | Add redirect URI to Azure AD app registration |
| `AADSTS7000215` | Invalid client secret | Generate new client secret in Azure Portal |
| `AADSTS700016` | Application not found | Check client ID matches app registration |
| `AADSTS90002` | Tenant not found | Check tenant ID is correct |

## Quick Fix Checklist

- [ ] Verify client secret is not expired (Azure Portal)
- [ ] Add Vercel redirect URI to Azure AD (Azure Portal → Authentication)
- [ ] Set all environment variables in Vercel (Vercel Dashboard → Settings)
- [ ] Redeploy Vercel app after changing environment variables
- [ ] Test authentication flow on production domain
- [ ] Check Vercel logs for configuration status
- [ ] Verify AUTH_URL matches your production domain exactly

## Still Having Issues?

1. **Check Vercel Logs**: Vercel Dashboard → Deployments → [Latest] → Runtime Logs
2. **Enable Debug Mode**: Add `DEBUG=1` environment variable in Vercel
3. **Test Locally**: Set environment variables in `.env.local` and run `npm run dev`
4. **Compare Configs**: Ensure local working config matches Vercel config exactly

## Additional Resources

- [NextAuth.js Deployment Guide](https://authjs.dev/getting-started/deployment)
- [Azure AD OAuth Provider Setup](https://authjs.dev/getting-started/providers/azure-ad)
- [Vercel Environment Variables](https://vercel.com/docs/projects/environment-variables)
- [Azure AD App Registration Guide](https://learn.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app)
