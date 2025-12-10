## Flow Logs (Dataverse Elastic Table) Query Support

This application now surfaces Power Platform Flow Log (table logical name: `flowlog`) data via a lightweight API endpoint and UI modal in the Power Platform Environments page.

### API Endpoint

`GET /api/power-platform/dataverse/flowlogs?instanceUrl={ORG_URL}&$top=50&partitionId={optional}&type={optional}`

Query parameters:

| Name | Required | Description |
| ---- | -------- | ----------- |
| `instanceUrl` | Yes | Base Dataverse org URL for the target environment (e.g. `https://org.crm.dynamics.com`). |
| `$top` | No | Maximum number of records to return (default 50). |
| `partitionId` | No | Elastic table partition hint for performance. If supplied it is passed using the capital `partitionId` query parameter per elastic table guidance. |
| `type` | No | Numeric flow log type discriminator. When provided adds a simple `type eq {value}` filter. |
| `$filter` | No | Raw OData filter applied verbatim (overrides `type`). |
| `$orderby` | No | OData order expression (defaults to `createdon desc`, auto‑fallback to `starttime desc` if `createdon` is invalid). |

Response shape:

```jsonc
{
  "partitionScoped": true,           // whether a partitionId hint was used
  "source": "<full request URL>",   // resolved query URL
  "value": [ { /* flowlog rows */ } ],
  "raw": { /* original Dataverse payload */ }
}
```

Errors:
- `404` with `error: "Flow Log table not found"` when the environment does not expose the `flowlog` table or the user/app lacks privileges.
- Other Dataverse errors pass through status codes with a simplified envelope.

### UI Usage

In the Power Platform Environments page, expand an environment and use the new **Flow Logs** button. The modal:

- Fetches the latest 50 records (no partition hint) sorted by `starttime`/`createdon`.
- Dynamically infers a compact column set: preferred columns if present, plus a few additional keys.
- Supports manual refresh (Refresh button) reusing the original instance URL.

### Notes & Future Enhancements

- Adding a partition selector (user / workflow partition) will significantly improve performance for large tenants.
- Support for server‑side filtering (date range, status) can be layered using `$filter` with additional UI controls.
- The API currently does not implement paging beyond `$top`; follow‑up could add `@odata.nextLink` handling if needed.

### Security & Permissions

The service principal / application user must have read privileges to the Flow Log (`flowlog`) table. If you receive 403/404 responses ensure:
1. The environment has automation analytics / flow logging features enabled.
2. The app registration is added as an Application User.
3. A security role granting read access to the Flow Log table is assigned.

### Elastic Table Considerations

Elastic tables benefit from partition‑scoped queries. Supply `partitionId=...` where possible (capital I) to avoid cross‑partition scans and achieve better latency. Without it, small `$top` values are recommended.
