# Pulse 360° – Quick Start

A trimmed, copy‑paste friendly path to get the portal running locally. For full detail (screenshots, rationale, troubleshooting) see `application-installation.md`.

---
## 1. Prerequisites (Verify)
- Node.js 18+ (LTS)
- Azure (Entra ID) tenant access to create an App Registration
- OpenSSL (comes with Git for Windows)

```powershell
node -v
npm -v
```

---
## 2. Clone
```powershell
git clone https://github.com/<your-user-or-org>/PortalofPortals.git
cd PortalofPortals
npm install
```

---
## 3. Register App (Choose ONE method)
### A) Portal (Fast UI)
1. Azure Portal → Entra ID → App registrations → New registration
2. Name: Pulse360Local (any) | Single tenant
3. Redirect URI (web): http://localhost:3000/api/auth/callback/azure-ad
4. Register → Copy Application (client) ID + Directory (tenant) ID
5. Certificates & secrets → New client secret → copy Value
6. API permissions → Add → Microsoft Graph (Application) → add (grouped for clarity):
  - Core: User.Read.All Group.Read.All Application.Read.All Device.Read.All Directory.Read.All AuditLog.Read.All SecurityEvents.Read.All
  - Risk & Identity: IdentityRiskyUser.Read.All
  - SharePoint (Sites pages): Sites.Read.All
  - Reporting (usage & metrics pages): Reports.Read.All
  - (Optional – Conditional Access metrics if 403): Policy.Read.All
7. Grant admin consent (repeat after adding any future optional permissions)
8. Remove unused permissions to enforce least privilege.

### B) CLI (Scriptable)
```powershell
$displayName = "Pulse360Local"
az ad app create --display-name $displayName --web-redirect-uris "http://localhost:3000/api/auth/callback/azure-ad"
$appId = az ad app list --display-name $displayName --query "[0].appId" -o tsv
$tenantId = az account show --query tenantId -o tsv
# Add Graph application permissions (core + feature sets) — omit lines you don't need
az ad app permission add --id $appId --api 00000003-0000-0000-c000-000000000000 `
  --api-permissions \
  User.Read.All=Role \
  Group.Read.All=Role \
  Application.Read.All=Role \
  Device.Read.All=Role \
  Directory.Read.All=Role \
  AuditLog.Read.All=Role \
  SecurityEvents.Read.All=Role \
  IdentityRiskyUser.Read.All=Role \
  Sites.Read.All=Role \
  Reports.Read.All=Role

# (Uncomment only if conditional access metrics require it)
# az ad app permission add --id $appId --api 00000003-0000-0000-c000-000000000000 --api-permissions Policy.Read.All=Role

# Grant admin consent after all additions
az ad app permission admin-consent --id $appId
# Create secret (1 year)
az ad app credential reset --id $appId --append --display-name "local-dev" --years 1
```
Record: appId (client), tenantId, generated secret value.

---
## 4. Environment File
Create `.env.local` (secret stays local):
```bash
AZURE_TENANT_ID=<tenant-guid>
AZURE_CLIENT_ID=<app-id>
AZURE_CLIENT_SECRET=<secret>
NEXTAUTH_URL=http://localhost:3000
```
---
## 5. Run
```powershell
npm run dev
```
Open http://localhost:3000 and log in with demo creds: `russdemo` / `russdemo`.

Dark mode is default. Override via `/?theme=light` or `/?theme=dark`.

---
## 6. Build (Prod Test)
```powershell
npm run build
npm start
```

---
## 7. Quick Troubleshooting
| Issue | Fix |
|-------|-----|
| 401 Graph calls | Re-grant admin consent / permissions mismatch |
| Login fails | Use demo creds or implement real NextAuth/MSAL |
| Env vars ignored | Restart dev; ensure file named `.env.local` |
| Light flash | Don’t remove early theme script in `layout.tsx` |

---
## 8. Next Steps (Optional)
- Swap demo auth → real NextAuth + MSAL (Authorization Code + PKCE)
- Add role-based authorization (App Roles)
- Move secrets to Key Vault in production
- Add CI (lint, test, Playwright screenshots)

---
**Need more depth?** Open `docs/application-installation.md`.
