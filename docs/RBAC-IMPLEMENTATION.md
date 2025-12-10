# Role-Based Access Control (RBAC) Implementation

## Overview

This document describes the role-based access control system implemented in the Portal of Portals application. The system provides two distinct user roles:

- **Admin**: Full access to all features including destructive operations (create, delete, update)
- **Read-only**: View-only access with no ability to perform destructive operations

## User Role Assignment

Users are automatically assigned roles based on their email address during authentication:

### Role Mapping

| Username (before @) | Role | Capabilities |
|-------------------|------|--------------|
| `russdemo` | `readonly` | View-only access, all destructive operations blocked |
| `russadmin` (or any other user) | `admin` | Full control including create/delete/update operations |

### Implementation Location

Role assignment is handled in `src/lib/auth.ts` in the JWT callback:

```typescript
// Extract username from email
const userEmail = (profile.email || profile.userPrincipalName || '') as string
const username = userEmail.toLowerCase().split('@')[0]

// Assign role based on username
if (username === 'russdemo') {
  token.role = 'readonly'
} else {
  token.role = 'admin'
}
```

The role is then passed to the session and made available throughout the application.

## Permission System

### Utility Functions

Location: `src/lib/permissions.ts`

The permission system provides utility functions for checking user capabilities:

```typescript
// Check if user can delete resources
canDelete(session: Session | null): boolean

// Check if user can create resources
canCreate(session: Session | null): boolean

// Check if user can update resources
canUpdate(session: Session | null): boolean

// Check if user is in read-only mode
isReadOnly(session: Session | null): boolean

// Get user-friendly denial message
getReadOnlyMessage(): string
```

### Usage Pattern

1. Import the permission utilities and NextAuth session:
```typescript
import { useSession } from 'next-auth/react'
import { canDelete, canCreate, getReadOnlyMessage } from '@/lib/permissions'
```

2. Get the user session:
```typescript
const { data: session } = useSession()
```

3. Use permission checks in UI components:
```typescript
// Disable create button for read-only users
<button
  onClick={() => canCreate(session) ? handleCreate() : alert(getReadOnlyMessage())}
  disabled={!canCreate(session)}
  className={canCreate(session) ? 'enabled-style' : 'disabled-style'}
>
  Create Resource
</button>

// Conditionally render delete section
{canDelete(session) && (
  <div className="danger-zone">
    <button onClick={handleDelete}>Delete</button>
  </div>
)}
```

## Protected Features

### Frontend Protection

The following UI components have been updated with role-based permission checks:

#### 1. Teams Management (`src/app/teams/list-teams/page.jsx`)

**Create Team Button (Line ~229)**
- Disabled for read-only users
- Shows alert with message when clicked by read-only user
- Visual styling changes based on permission

**Delete Team Button (Lines ~717-767)**
- Entire delete section only rendered for admin users
- Hidden completely from read-only users

#### 2. Applications Management (`src/app/applications/ApplicationsTable.jsx`)

**Delete Secret Button (Lines ~683-699)**
- Shows disabled trash icon for read-only users
- Active trash icon with delete functionality for admin users
- Displays permission message on hover for read-only users

#### 3. Power Platform Environments (`src/app/power-platform/environments/page.jsx`)

**Delete Environment Section (Lines ~2996-3033)**
- Entire "Danger Zone" section only rendered for admin users
- Hidden completely from read-only users
- Includes permanent environment deletion with confirmation

**Remove Team Member Button (Lines ~3672-3699)**
- Disabled for read-only users
- Visual indication (opacity + cursor) when disabled
- Tooltip explains permission requirement

### Backend Protection

All destructive API endpoints have been protected with server-side permission checks:

#### 1. Create Team (`src/app/api/teams/create-team/route.js`)

```javascript
// Check if user has permission to create
if (session.user.role === 'readonly') {
  return Response.json(
    { error: 'Permission denied: Read-only users cannot create teams' },
    { status: 403 }
  )
}
```

#### 2. Delete Team (`src/app/api/teams/delete-team/[id]/route.js`)

```javascript
// Check if user has permission to delete
if (session.user?.role === 'readonly') {
  return Response.json(
    { error: 'Permission denied: Read-only users cannot delete teams' },
    { status: 403 }
  )
}
```

#### 3. Delete Environment (`src/app/api/power-platform/environments/[environmentId]/delete/route.js`)

```javascript
// Check if user has permission to delete
if (session.user?.role === 'readonly') {
  return new Response(JSON.stringify({ 
    error: 'Permission denied: Read-only users cannot delete environments' 
  }), {
    status: 403,
    headers: { 'Content-Type': 'application/json' }
  })
}
```

#### 4. Remove Application Secret (`src/app/api/applications/[id]/remove-password/route.js`)

```javascript
// Check if user has permission to delete
if (session.user.role === 'readonly') {
  return NextResponse.json(
    { error: 'Permission denied: Read-only users cannot delete application secrets' },
    { status: 403 }
  )
}
```

## Security Considerations

### Defense in Depth

The RBAC system implements defense in depth with multiple layers of protection:

1. **Frontend UI Controls**: Buttons are disabled or hidden based on permissions
2. **API Route Guards**: Server-side checks reject unauthorized requests
3. **Session Validation**: All protected routes verify user session and role

### Why Both Frontend and Backend Protection?

- **Frontend**: Provides better UX by preventing users from attempting unauthorized actions
- **Backend**: Critical security layer that cannot be bypassed, even if frontend is modified

### Important Notes

⚠️ **Never rely solely on frontend checks for security**. Frontend code can be modified by users. The backend API protection is the actual security boundary.

## Testing the RBAC System

### As Admin User (russadmin)

1. Sign in with `russadmin` credentials
2. Verify you can:
   - Create teams
   - Delete teams
   - Delete environments
   - Remove application secrets
   - See all "Danger Zone" sections

### As Read-Only User (russdemo)

1. Sign in with `russdemo` credentials
2. Verify you:
   - Can view all resources (teams, environments, applications)
   - Cannot create teams (button disabled)
   - Cannot see delete team buttons (hidden)
   - Cannot see delete environment sections (hidden)
   - See disabled secret delete buttons
   - Get permission denied errors (403) if attempting API calls directly

### Testing API Protection

You can test API-level protection using curl or Postman:

```bash
# This should return 403 for russdemo, succeed for russadmin
curl -X DELETE https://your-domain.com/api/teams/delete-team/[team-id] \
  -H "Cookie: your-session-cookie"
```

## Adding New Protected Features

To add RBAC protection to new features:

### 1. Update Frontend Component

```typescript
import { useSession } from 'next-auth/react'
import { canDelete, getReadOnlyMessage } from '@/lib/permissions'

function MyComponent() {
  const { data: session } = useSession()
  
  return (
    <>
      {canDelete(session) && (
        <button onClick={handleDelete}>
          Delete Resource
        </button>
      )}
    </>
  )
}
```

### 2. Protect API Route

```javascript
import { auth } from '@/lib/auth'

export async function DELETE(request) {
  const session = await auth()
  
  if (!session?.user) {
    return Response.json({ error: 'Not authenticated' }, { status: 401 })
  }
  
  if (session.user.role === 'readonly') {
    return Response.json(
      { error: 'Permission denied: Read-only users cannot delete resources' },
      { status: 403 }
    )
  }
  
  // Proceed with deletion
}
```

## Customizing Role Assignment

To modify which users get which roles, update `src/lib/auth.ts`:

```typescript
// Example: Add more read-only users
const readOnlyUsers = ['russdemo', 'guest', 'demo']
if (readOnlyUsers.includes(username)) {
  token.role = 'readonly'
} else {
  token.role = 'admin'
}

// Example: Use a different attribute (e.g., group membership)
if (profile.groups?.includes('ReadOnlyGroup')) {
  token.role = 'readonly'
} else {
  token.role = 'admin'
}
```

## Future Enhancements

Potential improvements to the RBAC system:

1. **Additional Roles**: Add more granular roles (e.g., "operator", "auditor")
2. **Permission Sets**: Define specific permissions rather than broad role capabilities
3. **Resource-Level Permissions**: Allow/deny access to specific resources
4. **Azure AD Group Integration**: Derive roles from Azure AD group membership
5. **Admin UI**: Provide interface for managing user roles
6. **Audit Logging**: Track all permission checks and denied operations

## Troubleshooting

### Issue: User has wrong role

**Check:**
1. Verify the username is extracted correctly from email
2. Check the JWT token contains the role property
3. Verify session includes user.role

**Debug:**
```typescript
console.log('User email:', session.user?.email)
console.log('User role:', session.user?.role)
```

### Issue: API returns 403 for admin user

**Check:**
1. Session is being passed correctly to API route
2. Role is present in session.user.role
3. API route is checking the correct role property

### Issue: Frontend shows controls but API denies

This is **expected behavior** and demonstrates defense in depth. Frontend controls can be bypassed, so API must always enforce permissions.

## References

- NextAuth.js Documentation: https://next-auth.js.org/
- Next.js API Routes: https://nextjs.org/docs/api-routes/introduction
- JWT Best Practices: https://tools.ietf.org/html/rfc8725
