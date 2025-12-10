# AI Foundry Projects Implementation

**Date:** 2025-01-30
**Feature:** List Projects menu item in AI Foundry section
**Status:** ✅ Complete

## Overview

Added "List Projects" functionality to the AI Foundry menu, allowing users to query and view AI model deployments from Azure AI Foundry projects.

## Implementation Details

### 1. Menu Structure Update

**File:** `src/app/graph-explorer/nav-areas.ts`

Added new menu item to the AI Foundry `children` array:

```typescript
{
  name: 'AI Foundry', 
  href: '/azure/ai-foundry', 
  description: 'AI agents and projects', 
  icon: AcademicCapIcon,
  children: [
    { name: 'List Agents', href: '/azure/ai-foundry/agents', description: 'View AI Foundry agents', icon: UserIcon },
    { name: 'List Projects', href: '/azure/ai-foundry/projects', description: 'View AI Foundry model deployments', icon: FolderStackIcon },
  ]
}
```

**Icon:** Used `FolderStackIcon` to visually represent projects/deployments

### 2. Frontend Page

**File:** `src/app/azure/ai-foundry/projects/page.jsx`

**Features:**
- Client-side React component with form inputs for:
  - Project endpoint URL (required)
  - Model publisher filter (optional)
  - Model name filter (optional)
- Real-time data fetching with loading states and error handling
- Responsive table display showing:
  - Deployment name/ID
  - Model name and version
  - Model publisher
  - Status with color-coded badges (success/failed/in-progress)
  - Creation date
- Dark mode support with Tailwind styling
- Empty state messaging
- Input validation with helpful placeholder text

**Pattern:** Follows the same structure as the existing agents page for consistency

### 3. Backend API Route

**File:** `src/app/api/azure/ai-foundry/projects/route.js`

**Endpoint:** `GET /api/azure/ai-foundry/projects`

**Query Parameters:**
- `endpoint` (required): AI Foundry project endpoint URL
  - Format: `https://<aiservices-id>.services.ai.azure.com/api/projects/<project-name>`
- `modelPublisher` (optional): Filter results by model publisher
- `modelName` (optional): Filter results by model name

**Implementation:**
1. **Authentication:**
   - Validates NextAuth session
   - Uses `ClientSecretCredential` with service principal
   - Obtains Azure access token for scope `https://ai.azure.com/.default`

2. **API Call:**
   - Constructs URL: `{endpoint}/deployments?api-version=v1&deploymentType=ModelDeployment&modelPublisher={mp}&modelName={mn}`
   - Sends authenticated request with Bearer token
   - Handles errors (401, 403, 500) with detailed logging

3. **Response:**
   - Returns JSON data with `data` array of deployments
   - Includes error details for troubleshooting

**Security:**
- Validates endpoint format to ensure proper Azure AI Foundry URL structure
- Uses environment variables for credentials (AZURE_TENANT_ID, AZURE_CLIENT_ID, AZURE_CLIENT_SECRET)
- Checks session before any API calls

## API Endpoint Format

The Azure AI Foundry deployments endpoint follows this structure:

```
GET {endpoint}/deployments?api-version=v1&modelPublisher={modelPublisher}&modelName={modelName}&deploymentType=ModelDeployment
```

**Example:**
```
GET https://my-ai-service.services.ai.azure.com/api/projects/my-project/deployments?api-version=v1&deploymentType=ModelDeployment
```

## Environment Variables Required

```env
AZURE_TENANT_ID=<your-tenant-id>
AZURE_CLIENT_ID=<your-client-id>
AZURE_CLIENT_SECRET=<your-client-secret>
```

## Testing

### Build Verification
```bash
npm run build
```
✅ No compilation errors

### Files Checked
- ✅ `src/app/graph-explorer/nav-areas.ts` - No errors
- ✅ `src/app/azure/ai-foundry/projects/page.jsx` - No errors
- ✅ `src/app/api/azure/ai-foundry/projects/route.js` - No errors

### Manual Testing Steps

1. **Navigation:**
   - Open application
   - Navigate to Azure section
   - Expand "AI Foundry" menu
   - Click "List Projects"

2. **Basic Functionality:**
   - Enter a valid project endpoint URL
   - Click "Fetch Deployments"
   - Verify loading state appears
   - Verify results display in table

3. **Filtering:**
   - Enter model publisher (e.g., "OpenAI")
   - Click "Fetch Deployments"
   - Verify filtered results

4. **Error Handling:**
   - Test with invalid endpoint URL
   - Test with missing endpoint
   - Verify error messages display correctly

5. **Responsive Design:**
   - Test on different screen sizes
   - Verify table scrolls horizontally on mobile
   - Check dark mode appearance

## Code Quality

### Patterns Followed
1. **Consistency:** Matches existing agents implementation pattern
2. **Error Handling:** Comprehensive try/catch with detailed logging
3. **Validation:** Input validation on both frontend and backend
4. **Styling:** Uses Tailwind with dark mode support
5. **Security:** Session authentication + credential validation

### Documentation
- Inline JSDoc comments in API route
- Helpful placeholder text in UI
- Error messages guide user to correct issues

## Files Created/Modified

**Created:**
1. `src/app/azure/ai-foundry/projects/page.jsx` - Frontend UI component
2. `src/app/api/azure/ai-foundry/projects/route.js` - Backend API route
3. `docs/ai-foundry-projects-implementation.md` - This documentation

**Modified:**
1. `src/app/graph-explorer/nav-areas.ts` - Added menu item to AI Foundry section

## Future Enhancements

Potential improvements for future iterations:

1. **Pagination:** Add support for large result sets
2. **Sorting:** Allow users to sort by different columns
3. **Detailed View:** Click deployment to see full details
4. **Refresh:** Auto-refresh or manual refresh button
5. **Export:** Export deployment list to CSV/JSON
6. **Search:** Client-side search within results
7. **Filters:** Additional filters (status, date range, etc.)
8. **Caching:** Cache results with SWR for better performance

## Related Documentation

- [AI Foundry Agents Implementation](./copilot-studio-agents-botcomponent-enhancement.md) - Similar implementation pattern
- [Azure API Integration](./API-DOCUMENTATION.md) - General Azure API patterns
- [Environment Variables](./ENVIRONMENT-VARIABLES.md) - Required configuration

## Completion Checklist

- ✅ Menu item added to navigation
- ✅ Frontend page created with form and table
- ✅ Backend API route implemented
- ✅ Authentication and authorization implemented
- ✅ Error handling added
- ✅ Input validation implemented
- ✅ Dark mode styling applied
- ✅ No compilation errors
- ✅ Build passes successfully
- ✅ Documentation created
