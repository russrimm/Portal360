# Fixing "invalid_client" OAuth Error on Vercel

## Problem
You're seeing this error on Vercel (but not localhost):
```
OAuthCallbackError: OAuth Provider returned an error: invalid_client
```

This means Azure AD is rejecting the authentication attempt from your Vercel deployment.

## Root Causes & Solutions

### 1. **Redirect URI Not Configured in Azure AD** ⚠️ MOST COMMON

Azure AD needs to know which URLs can receive OAuth callbacks.

**Fix:**
1. Go to [Azure Portal](https://portal.azure.com)
2. Navigate to: **Azure Active Directory** → **App registrations** → [Your App]
3. Click **Authentication** in the left menu
4. Under **Redirect URIs**, add:
   - `https://portalofportals.vercel.app/api/auth/callback/azure-ad`
   - If you have preview deployments, also add: `https://*.vercel.app/api/auth/callback/azure-ad` (or add each preview URL individually)
5. Click **Save**

**Important:** The callback URL must EXACTLY match:
- Protocol: `https://` (not `http://`)
- Domain: Your Vercel deployment URL
- Path: `/api/auth/callback/azure-ad`

---

### 2. **Environment Variables Not Set in Vercel**

**Fix:**
1. Go to your Vercel project: https://vercel.com/[your-account]/portalofportals
2. Click **Settings** → **Environment Variables**
3. Add these variables (for **Production**, **Preview**, and **Development**):

```bash
# Required - Azure AD App Registration
AZURE_CLIENT_ID=your-client-id-here
AZURE_CLIENT_SECRET=your-client-secret-here
AZURE_TENANT_ID=your-tenant-id-here

# Required - NextAuth Configuration
AUTH_SECRET=generate-a-random-string-here
AUTH_URL=https://portalofportals.vercel.app

# Optional - Alternative variable names (NextAuth v5)
AUTH_AZURE_AD_CLIENT_ID=same-as-AZURE_CLIENT_ID
AUTH_AZURE_AD_CLIENT_SECRET=same-as-AZURE_CLIENT_SECRET
AUTH_AZURE_AD_TENANT_ID=same-as-AZURE_TENANT_ID
```

**To generate AUTH_SECRET:**
```bash
openssl rand -base64 32
```

4. **Redeploy** your application after adding variables

---

### 3. **Client Secret Expired or Incorrect**

Azure AD client secrets expire (typically after 6, 12, or 24 months).

**Check:**
1. Go to **Azure Portal** → **App registrations** → [Your App]
2. Click **Certificates & secrets**
3. Check if your secret has expired
4. If expired or unsure, create a new one:
   - Click **New client secret**
   - Choose expiration period
   - Copy the **Value** (not the Secret ID!)
   - Update `AZURE_CLIENT_SECRET` in Vercel

⚠️ **Warning:** Copy the secret value immediately - you can't view it again after leaving the page!

---

### 4. **Client ID Incorrect**

**Verify:**
1. Go to **Azure Portal** → **App registrations** → [Your App]
2. Copy the **Application (client) ID** from the Overview page
3. Verify it matches `AZURE_CLIENT_ID` in Vercel (should be a GUID like `12345678-1234-1234-1234-123456789abc`)

---

## Testing the Fix

1. After making changes in Azure AD, wait 1-2 minutes for propagation
2. Redeploy your Vercel app (or wait for auto-deploy)
3. Go to: `https://portalofportals.vercel.app`
4. Click sign in
5. Should redirect to Microsoft login successfully

---

## Debugging Steps

If still having issues, check Vercel logs:

1. Go to Vercel Dashboard → Your Project → Deployments
2. Click on your latest deployment
3. Click **Functions** tab
4. Look at `/api/auth/[...nextauth]` logs
5. You should see a log line like:
   ```
   [auth] Production configuration: {
     hasClientId: true,
     clientIdPrefix: 'abcd1234...',
     hasClientSecret: true,
     hasTenantId: true,
     hasSecret: true,
     authUrl: 'https://portalofportals.vercel.app',
     issuer: 'https://login.microsoftonline.com/your-tenant-id/v2.0'
   }
   ```

If any values are `false` or `'not set'`, that variable is missing in Vercel.

---

## Quick Checklist

- [ ] Redirect URI added in Azure AD Authentication settings
- [ ] All environment variables set in Vercel (Production, Preview, Development)
- [ ] Client secret is valid and not expired
- [ ] Client ID matches Azure AD application
- [ ] AUTH_URL matches your Vercel production domain
- [ ] AUTH_SECRET is set (random string)
- [ ] Redeployed after making changes

---

## Additional Notes

- **Preview Deployments:** Each preview deployment gets a unique URL. You may need to add wildcard redirect URIs (`*.vercel.app`) or add each preview URL individually.
- **Custom Domains:** If using a custom domain, add redirect URIs for that domain too.
- **Multiple Environments:** Set environment variables separately for Production, Preview, and Development in Vercel.

---

## Still Having Issues?

1. Check if you can sign in on localhost (rules out Azure AD configuration)
2. Compare environment variables between local `.env.local` and Vercel
3. Check Azure AD sign-in logs: **Azure Portal** → **Azure AD** → **Sign-in logs**
4. Look for error codes and descriptions in the logs
