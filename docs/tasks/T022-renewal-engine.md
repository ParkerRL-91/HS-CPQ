# T022 — Renewal Engine

**Sprint:** [Sprint 05](../sprints/sprint-05.md) | **Phase:** 3

## User Stories
- As the system, renewal quotes are automatically generated at configurable lead times (90/60/30 days) before contract expiry.
- As a rep, the renewal quote is pre-populated with the current subscription terms so I don't need to rebuild it from scratch.
- As a manager, I can configure the renewal pricing method (carry forward, list price, list + uplift, custom) per product or globally.

## Definition of Success
- Daily Inngest job scans for subscriptions expiring within 90, 60, and 30 days and generates renewal quotes at the first matching window.
- Renewal quote is pre-populated with the subscription's product, quantity, and the renewal pricing method result.
- Renewal quote is assigned to the deal owner and surfaced in the HubSpot Deal as an associated quote.
- On acceptance of the renewal quote, the `Subscription` record is updated with new term dates and a new HubSpot Deal is created.
- Renewal pricing methods: carry forward (same contracted price), list price, list + uplift (configurable %), custom (rep sets price).

## Files & Objects Touched
- `packages/subscriptions/src/renewal-engine.ts` — renewal quote generation logic
- `packages/subscriptions/src/renewal-pricer.ts` — renewal pricing method dispatch
- `packages/jobs/src/renewal-scan.ts` — daily Inngest job
- `packages/db/prisma/schema.prisma` — `Subscription.renewal_quote_id`, `Subscription.renewal_method`
- `packages/hubspot/sync/renewal-deal-creator.ts` — creates new HubSpot Deal on renewal

## Methodology
1. Define `renewal-scan` Inngest job: runs daily at 6am UTC; queries subscriptions where `end_date` is within 90, 60, or 30 days and `renewal_quote_id IS NULL` for the applicable window.
2. `renewal-engine.ts`: for each identified subscription, calls `renewal-pricer.ts` to compute the renewal unit price, creates a new `Quote` (type = renewal) with the subscription's product/quantity/term, links it to the subscription's `hubspot_deal_id`.
3. `renewal-pricer.ts`: switch on `subscription.renewal_method`; carry forward = `subscription.contracted_price`; list price = current `product.list_price`; list + uplift = `product.list_price * (1 + uplift_pct)`; custom = `null` (rep must enter).
4. Write the renewal quote back to HubSpot as an associated quote on the deal via the CRM API.
5. On renewal quote acceptance: update `Subscription.start_date`, `end_date`, `contracted_price`; call `renewal-deal-creator.ts` to create a new HubSpot Deal linked to the company and contact, representing the renewal revenue.
6. Prevent duplicate renewal quote generation: check `renewal_quote_id IS NOT NULL` before creating; use an Inngest idempotency key per `subscription_id + renewal_window`.
