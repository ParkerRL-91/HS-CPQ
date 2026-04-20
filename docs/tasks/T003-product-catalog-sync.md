# T003 — Product Catalog Sync from HubSpot

**Sprint:** [Sprint 01](../sprints/sprint-01.md) | **Phase:** 1

## User Stories
- As a sales rep, when I open the quote builder I see all my company's products from HubSpot, so I don't need to maintain a separate product list.
- As an admin, I can enrich each HubSpot product with CPQ-specific attributes (pricing method, floor price) without losing the connection to the source record.
- As the system, changes to products in HubSpot are reflected in the CPQ within a configurable sync interval.

## Definition of Success
- All HubSpot products for a portal are synced to the `Product` table, keyed by `hubspot_product_id`.
- CPQ extension fields (`pricing_method`, `floor_price`, `usage_model`, `dimensions`) are preserved on upsert.
- Sync runs on app install (full sync) and on a scheduled interval (incremental, using `updatedAt` filter).
- HubSpot webhook on `product.propertyChange` triggers an incremental sync for the changed record.
- Sync errors are logged with product ID, error, and retry count; failed records are retried with exponential backoff.

## Files & Objects Touched
- `packages/hubspot/sync/product-sync.ts` — full and incremental sync logic
- `packages/db/prisma/schema.prisma` — `Product` model CPQ extension fields
- `apps/web/app/api/webhooks/hubspot/route.ts` — webhook handler routing product change events
- `apps/web/app/api/admin/sync/route.ts` — manual sync trigger endpoint
- `packages/jobs/` — scheduled sync job definition (Inngest)

## Methodology
1. Implement `fullSync(portalId)`: pages through HubSpot Products API (`/crm/v3/objects/products`), upserts each record into the `Product` table using `hubspot_product_id` as the conflict key, preserving any existing CPQ extension fields.
2. Implement `incrementalSync(portalId, since)`: fetches products with `updatedAt > since`, performs same upsert logic.
3. Register a HubSpot CRM webhook subscription for `product.propertyChange`; the webhook handler parses the payload, identifies the changed product, and queues an incremental sync job.
4. Define an Inngest scheduled job that runs `incrementalSync` every 15 minutes per active portal.
5. Expose `POST /api/admin/sync` (admin-only) for on-demand full sync.
6. Add retry logic: failed products are requeued with exponential backoff (1m → 5m → 30m → give up + alert).
