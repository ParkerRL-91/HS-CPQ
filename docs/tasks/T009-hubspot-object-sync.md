# T009 — HubSpot Object Sync

**Sprint:** [Sprint 02](../sprints/sprint-02.md) | **Phase:** 1

## User Stories
- As a sales manager, I can see CPQ quote data (status, value, discount %) on the HubSpot Deal record so that HubSpot reports reflect real quote activity.
- As the system, when a quote is accepted the HubSpot Deal stage is automatically updated so that the pipeline is always current.
- As a rep, line items I configure in CPQ are visible as HubSpot line items on the deal.

## Definition of Success
- On every quote status change, `Quote.status`, `Quote.total`, and `Quote.discount_pct` are written to the corresponding HubSpot Deal custom properties.
- On acceptance, HubSpot Deal stage is updated to the configured "Closed Won" stage.
- CPQ line items (product, quantity, unit price, discount, total) are synced to HubSpot Line Item objects associated with the Deal.
- HubSpot Quote object is created/updated with `pdf_url`, `expiry_date`, and `status`.
- Sync errors are retried with backoff and logged; a dead-letter alert fires after 3 consecutive failures.

## Files & Objects Touched
- `packages/hubspot/sync/quote-sync.ts` — write-back logic for quotes, line items, deal properties
- `packages/hubspot/sync/deal-sync.ts` — deal stage update
- `packages/jobs/src/sync-to-hubspot.ts` — Inngest job triggered on quote events
- `packages/api/src/routers/quote.ts` — triggers sync job on relevant mutations
- `packages/hubspot/client.ts` — CRM API wrapper used by sync modules

## Methodology
1. Define the set of HubSpot Deal custom properties to create via the Properties API on app install: `cpq_quote_status`, `cpq_quote_total`, `cpq_quote_discount_pct`, `cpq_renewal_date`.
2. Implement `quote-sync.ts`: given a `quoteId`, fetch full quote data, map to HubSpot property patch payloads, batch-update the Deal and native Quote object via HubSpot CRM API.
3. Implement line item sync: for each CPQ `LineItem`, upsert a HubSpot Line Item object (match on `hubspot_line_item_id`; create if missing); delete HubSpot line items no longer in the CPQ quote.
4. Implement `deal-sync.ts`: on `quote.status = accepted`, call the HubSpot Deals API to update `dealstage` to the configured Closed Won stage ID.
5. Define `sync-to-hubspot` Inngest job triggered by quote mutations; ensures sync is async and doesn't block the API response.
6. Implement retry with exponential backoff (3 retries); after 3 failures, log to Sentry and send an internal Slack alert.
