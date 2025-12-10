# Fixes for Authentication and API Issues (December 8, 2025)

## Issue Summary

Two critical production issues on Vercel deployment:

1. **404 Error on `/api/azure/reservation-summaries`** - Route not being found
2. **AADSTS9002326 Cross-Origin Token Redemption Error** - Authentication failing with "Cross-origin token redemption is permitted only for the 'Single-Page Application' client-type"

---

## Issue 1: Reservation Summaries 404 Error

### Problem
The `/api/azure/reservation-summaries` endpoint exists in the codebase at `src/app/api/azure/reservation-summaries/route.js` but returns 404 in production.

### Root Cause Analysis
This is likely caused by one of the following:

1. **Build Output Issue**: The route file may not be included in the production build
2. **Dynamic Rendering**: The route uses `export const dynamic = 'force-dynamic'` which requires proper Node.js runtime configuration
3. **Standalone Output Mode**: With `output: 'standalone'` in `next.config.ts`, some API routes may need explicit runtime configuration

### Solution

#### Step 1: Verify Route Configuration
Ensure the route has proper runtime configuration:

```javascript
// src/app/api/azure/reservation-summaries/route.js
export const dynamic = 'force-dynamic';
export const runtime = 'nodejs'; // Explicitly set Node.js runtime
```

#### Step 2: Check Build Output
After building, verify the route appears in `.next/server/app/api/azure/reservation-summaries/route.js`

```powershell
# Local build test
npm run build
Get-ChildItem -Path .next\server\app\api\azure\reservation-summaries -Recurse
```

#### Step 3: Verify Vercel Configuration
Ensure your `vercel.json` (if it exists) doesn't exclude API routes:

```json
{
  "functions": {
    "api/**/*.js": {
      "memory": 1024,
      "maxDuration": 30
    }
  }
}
```

### Implementation

**File: `src/app/api/azure/reservation-summaries/route.js`**

Add explicit runtime configuration:

```javascript
import { auth } from '@/lib/auth';
import { getAzureManagementToken } from '@/lib/azureToken';

export const dynamic = 'force-dynamic';
export const runtime = 'nodejs'; // ADD THIS LINE

// ... rest of the code
```

---

## Issue 2: AADSTS9002326 Cross-Origin Token Redemption Error

### Problem
When calling Azure APIs from the browser on Vercel (`https://portalofportals.vercel.app`), getting error:

```
AADSTS9002326: Cross-origin token redemption is permitted only for the 'Single-Page Application' client-type. 
Request origin: 'https://portalofportals.vercel.app'
```

### Root Cause
Your Azure AD app registration redirect URI for Vercel is configured as **"Web"** platform type, but Microsoft Entra ID is detecting cross-origin requests that require **"SPA"** platform type.

### Why This Happens

**Next.js Server-Side vs Client-Side Token Flow:**

1. **Server-Side (Correct - No CORS):**
   - User logs in → NextAuth server gets tokens
   - API routes use tokens server-side
   - No CORS issues because server-to-server calls

2. **Client-Side (Problem - CORS):**
   - JavaScript in browser tries to use tokens
   - Browser makes cross-origin request to Azure APIs
   - Azure detects cross-origin and requires SPA platform type

### The Issue with Current Architecture

Your app is configured as **"Web"** application type in Azure AD, which means:
- ✅ Server-side token acquisition works (NextAuth)
- ❌ Client-side token usage triggers CORS errors
- ❌ Cross-origin token redemption blocked

### Solution Options

You have **three approaches** to fix this:

---

#### **Option 1: Keep Web App Type + Server-Only API Calls (RECOMMENDED)**

**Best Practice for Next.js Apps**

Never call Azure APIs directly from the browser. All Azure API calls should go through your Next.js API routes.

**Implementation:**

1. **Ensure all Azure API calls go through your backend**

   **Bad (Client-side - causes CORS):**
   ```javascript
   // ❌ DON'T DO THIS - Direct browser call to Azure
   const response = await fetch('https://management.azure.com/...', {
     headers: { Authorization: `Bearer ${session.accessToken}` }
   });
   ```

   **Good (Server-side - no CORS):**
   ```javascript
   // ✅ DO THIS - Call your API route instead
   const response = await fetch('/api/azure/cost-details', {
     method: 'POST',
     headers: { 'Content-Type': 'application/json' },
     body: JSON.stringify({ scope, timePeriod })
   });
   ```

2. **Review all client components for direct Azure API calls**

   Search for patterns like:
   - `fetch('https://management.azure.com'`
   - `fetch('https://graph.microsoft.com'`
   - `fetch('https://api.powerplatform.com'`

   Replace with calls to your `/api/azure/*` routes.

3. **Add middleware to prevent direct Azure token usage from browser**

   **File: `src/middleware.ts`** (create if doesn't exist)

   ```typescript
   import { NextResponse } from 'next/server';
   import type { NextRequest } from 'next/server';

   export function middleware(request: NextRequest) {
     // Block direct Azure API calls from client
     const url = request.nextUrl.clone();
     const origin = request.headers.get('origin');
     
     if (origin && (
       url.hostname.includes('azure.com') ||
       url.hostname.includes('microsoft.com') ||
       url.hostname.includes('powerplatform.com')
     )) {
       return new NextResponse('Direct Azure API calls not allowed from browser', { 
         status: 403 
       });
     }
     
     return NextResponse.next();
   }

   export const config = {
     matcher: ['/((?!_next/static|_next/image|favicon.ico).*)']
   };
   ```

**Pros:**
- ✅ Most secure approach
- ✅ Tokens never exposed to browser
- ✅ No CORS issues
- ✅ No Azure AD configuration changes needed
- ✅ Better error handling and logging

**Cons:**
- ⚠️ Requires refactoring client-side Azure API calls

---

#### **Option 2: Add SPA Platform Redirect URI**

**Add Vercel URL as SPA platform type in addition to Web**

This allows both server-side (Web) and client-side (SPA) token usage.

**Steps:**

1. Go to [Azure Portal](https://portal.azure.com) → Azure AD → App Registrations → Your App
2. Click **Authentication** in left menu
3. Under **Platform configurations**, click **Add a platform**
4. Select **Single-page application**
5. Add redirect URI: `https://portalofportals.vercel.app/api/auth/callback/azure-ad`
6. Enable **Access tokens** and **ID tokens** checkboxes
7. Click **Configure**

**Important:** Keep your existing **Web** platform redirect URI - you need both!

**Pros:**
- ✅ Allows client-side token usage if needed
- ✅ No code changes required

**Cons:**
- ⚠️ Less secure (tokens accessible in browser)
- ⚠️ Still not best practice for server-rendered apps
- ⚠️ May expose tokens in browser DevTools

---

#### **Option 3: Migrate to Pure SPA Architecture**

**Convert entire app to SPA platform type**

This is **NOT RECOMMENDED** for Next.js server-rendered apps.

**Why Not:**
- ❌ Breaks server-side rendering benefits
- ❌ Requires rewriting authentication flow
- ❌ Loses NextAuth.js benefits
- ❌ More complex token management

---

## Recommended Implementation Plan

### Step 1: Fix Reservation Summaries Route (Quick Fix)
```javascript
// src/app/api/azure/reservation-summaries/route.js
export const runtime = 'nodejs'; // Add this line at top
```

### Step 2: Audit Client-Side Azure API Calls
```powershell
# Search for direct Azure API calls in client components
grep -r "fetch.*management.azure.com" src/
grep -r "fetch.*graph.microsoft.com" src/
grep -r "fetch.*powerplatform.com" src/
```

### Step 3: Find and Fix Cross-Origin Calls

Look for this pattern in client components:
```javascript
'use client'

// ❌ BAD - Direct Azure call from browser
const session = useSession();
const response = await fetch('https://management.azure.com/...', {
  headers: { Authorization: `Bearer ${session.data.accessToken}` }
});
```

Replace with:
```javascript
'use client'

// ✅ GOOD - Call through your API route
const response = await fetch('/api/azure/some-endpoint', {
  method: 'POST',
  body: JSON.stringify({ params })
});
```

### Step 4: Verify All API Routes Have Proper Token Handling

All API routes should:
1. Check authentication with `await auth()`
2. Use `getAzureManagementToken()` for Azure Management API
3. Use `getPowerPlatformDelegatedToken()` for Power Platform API
4. Never expose tokens to client responses

**Example Pattern:**
```javascript
// src/app/api/azure/[resource]/route.js
import { auth } from '@/lib/auth';
import { getAzureManagementToken } from '@/lib/azureToken';

export const dynamic = 'force-dynamic';
export const runtime = 'nodejs';

export async function GET(request) {
  const session = await auth();
  if (!session?.user) {
    return new Response(JSON.stringify({ error: 'Not authenticated' }), {
      status: 401,
      headers: { 'Content-Type': 'application/json' }
    });
  }

  try {
    // Get token server-side
    const accessToken = await getAzureManagementToken({
      userAccessToken: session.accessToken,
      refreshToken: session.refreshToken
    });

    // Call Azure API server-side
    const response = await fetch('https://management.azure.com/...', {
      headers: {
        'Authorization': `Bearer ${accessToken}`,
        'Content-Type': 'application/json'
      }
    });

    const data = await response.json();

    // Return data to client (NO TOKEN!)
    return new Response(JSON.stringify({ success: true, data }), {
      status: 200,
      headers: { 'Content-Type': 'application/json' }
    });

  } catch (error) {
    console.error('[API Error]:', error);
    return new Response(JSON.stringify({
      success: false,
      error: error.message
    }), {
      status: 500,
      headers: { 'Content-Type': 'application/json' }
    });
  }
}
```

---

## Testing Checklist

### Local Testing
```powershell
# 1. Build the app
npm run build

# 2. Start production server
npm start

# 3. Test reservation summaries endpoint
curl http://localhost:3000/api/azure/reservation-summaries

# 4. Test cost details endpoint
curl -X POST http://localhost:3000/api/azure/cost-details \
  -H "Content-Type: application/json" \
  -d '{"scope":"/subscriptions/YOUR_SUB_ID","timePeriod":{"from":"2025-12-01","to":"2025-12-08"}}'
```

### Vercel Testing
1. Deploy to Vercel: `git push origin main`
2. Check build logs for API route compilation
3. Test endpoints in browser DevTools Network tab
4. Verify no CORS errors in console

### Azure AD Verification
1. Go to Azure Portal → Azure AD → Enterprise Applications → Your App
2. Click **Sign-ins** in left menu
3. Look for recent sign-ins
4. Verify no "cross-origin" errors in failure logs

---

## Related Files Modified

- `src/app/api/azure/reservation-summaries/route.js` - Added runtime export
- `src/lib/auth.ts` - Already correctly configured for Web app type
- `src/lib/azureToken.js` - Already has proper OBO token refresh logic

---

## Prevention: Best Practices

### For Future API Routes

Always follow this pattern:

```javascript
// 1. Export runtime and dynamic
export const runtime = 'nodejs';
export const dynamic = 'force-dynamic';

// 2. Check authentication first
const session = await auth();
if (!session?.user) {
  return new Response(JSON.stringify({ error: 'Not authenticated' }), {
    status: 401,
    headers: { 'Content-Type': 'application/json' }
  });
}

// 3. Get tokens server-side only
const accessToken = await getAzureManagementToken({
  userAccessToken: session.accessToken,
  refreshToken: session.refreshToken
});

// 4. Make Azure API calls server-side
const response = await fetch('https://management.azure.com/...', {
  headers: { 'Authorization': `Bearer ${accessToken}` }
});

// 5. Return data only (never tokens!)
return new Response(JSON.stringify({ data }), {
  status: 200,
  headers: { 'Content-Type': 'application/json' }
});
```

### For Client Components

**Never do this:**
```javascript
'use client'
// ❌ BAD
const { data: session } = useSession();
fetch('https://management.azure.com/...', {
  headers: { Authorization: `Bearer ${session.accessToken}` }
});
```

**Always do this:**
```javascript
'use client'
// ✅ GOOD
fetch('/api/azure/my-endpoint', {
  method: 'POST',
  body: JSON.stringify({ params })
});
```

---

## Summary

**Immediate Actions:**
1. ✅ Add `export const runtime = 'nodejs'` to reservation-summaries route
2. ✅ Search codebase for direct Azure API calls from client components
3. ✅ Refactor any found calls to go through API routes
4. ✅ Deploy and test

**Azure AD Configuration:**
- ✅ Keep current "Web" platform type
- ❌ Do NOT change to SPA (not needed if following best practices)

**Long-term:**
- ✅ All Azure API calls must go through `/api/azure/*` routes
- ✅ Never expose tokens to browser
- ✅ Always use server-side token acquisition
