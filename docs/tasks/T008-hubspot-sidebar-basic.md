# T008 — HubSpot Sidebar (Basic)

**Sprint:** [Sprint 02](../sprints/sprint-02.md) | **Phase:** 1

## User Stories
- As a rep, I can see all quotes for the current deal directly in the HubSpot Deal sidebar without switching tabs.
- As a rep, I can create a new quote from the sidebar with one click and it opens pre-loaded with the deal's context.
- As a rep, I can see the current approval status of each quote from the sidebar.

## Definition of Success
- HubSpot UI Extension CRM card renders in the Deal record sidebar via the registered extension endpoint.
- Quote Summary Panel lists all quotes for the deal: status badge, quote value, created date, and action buttons (Open, Send, Delete).
- "Create New Quote" button launches the CPQ configurator (`/quotes/new?dealId=...`) in a new tab, pre-populated with deal/company/contact data.
- Approval status indicator shows current state and approver name when `status = pending_approval`.
- Extension authenticates using the portal's stored OAuth token, never exposing credentials client-side.

## Files & Objects Touched
- `apps/hubspot-extension/src/app.tsx` — root UI Extension component
- `apps/hubspot-extension/src/components/QuoteSummaryPanel.tsx`
- `apps/hubspot-extension/src/components/ApprovalStatusBadge.tsx`
- `apps/web/app/api/extensions/quotes/route.ts` — data endpoint called by the extension
- `hubspot.config.yml` — UI Extension registration
- `apps/web/app/quotes/new/page.tsx` — new quote page with deal context pre-loading

## Methodology
1. Register the UI Extension in `hubspot.config.yml` targeting the Deal record type; specify the extension card's API endpoint.
2. Build the extension React app using `@hubspot/ui-extensions-sdk`; on mount, call the portal's `/api/extensions/quotes` endpoint with the deal ID from the extension context.
3. `QuoteSummaryPanel` renders each quote as a card with status, value, timestamps, and buttons using HubSpot's `hubspot.ui` component primitives.
4. "Create New Quote" calls `hubspot.openExternalLink` to open the CPQ configurator; the link includes `dealId`, `companyId`, and `contactId` as query params.
5. `ApprovalStatusBadge` derives display state from `quote.status` and `quote.approval_request`; polls every 30 seconds for live updates.
6. The `/api/extensions/quotes` endpoint validates the request's HubSpot-signed header, fetches quotes for the portal/deal, and returns the serialized list.
