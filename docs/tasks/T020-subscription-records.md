# T020 — Subscription Records

**Sprint:** [Sprint 05](../sprints/sprint-05.md) | **Phase:** 3

## User Stories
- As the system, when a deal with recurring line items moves to Closed Won a subscription record is automatically created.
- As a rep, I can view all active subscriptions for an account in the CPQ platform.
- As a manager, subscription data drives MRR/ARR reporting rather than deal amounts.

## Definition of Success
- HubSpot webhook on Deal stage change to Closed Won triggers subscription creation for any line items with `billing_frequency != one_time`.
- `Subscription` record captures: product, quantity, contracted price, billing frequency, start/end date, auto-renewal flag, HubSpot deal ID and company ID.
- Subscription list view shows all active subscriptions per portal with filterable status.
- Subscriptions are linked back to their originating HubSpot Deal.

## Files & Objects Touched
- `packages/subscriptions/src/subscription-creator.ts` — creates subscriptions from a closed-won deal's line items
- `packages/db/prisma/schema.prisma` — `Subscription` model
- `apps/web/app/api/webhooks/hubspot/route.ts` — deal stage change handler
- `apps/web/app/subscriptions/page.tsx` — subscription list view
- `packages/api/src/routers/subscription.ts` — CRUD and list procedures

## Methodology
1. Register a HubSpot CRM webhook for `deal.propertyChange` on the `dealstage` property.
2. Webhook handler: when `dealstage` changes to the configured Closed Won stage ID, queues a `create-subscriptions` Inngest job with the deal ID.
3. `subscription-creator.ts`: fetches line items for the deal; for each line item where `billing_frequency != one_time`, creates a `Subscription` record using the CPQ line item's `unit_price`, `quantity`, `billing_frequency`, quote's `start_date` and `end_date`.
4. Mark the source `LineItem` with `subscription_id` so the subscription is traceable back to the quote.
5. Build subscription list page: table with company name, product, quantity, ARR value, status, start/end dates, days until renewal. Sortable and filterable by status.
6. Write tests covering: deal with mixed one-time and recurring line items (only recurring create subscriptions), deal with no recurring items (no subscriptions), duplicate webhook delivery (idempotent creation).
