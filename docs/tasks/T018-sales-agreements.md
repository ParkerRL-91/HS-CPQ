# T018 — Sales Agreements

**Sprint:** [Sprint 04](../sprints/sprint-04.md) | **Phase:** 2

## User Stories
- As an admin, I can create a Sales Agreement for a customer account specifying contracted prices for specific products.
- As a rep, when I create a quote for an account with an active Sales Agreement the contracted prices are automatically applied.
- As a rep, I cannot override a contracted price upward without triggering an approval.

## Definition of Success
- `SalesAgreement` is linked to a HubSpot company ID with start/end dates and status (active, expired, pending_approval, draft).
- `AgreementLineItem` records specify product, contracted price (or max discount %), and optional minimum commit quantity.
- At quote creation, the system checks for an active agreement for the deal's company; matching products have their price set to the contracted price automatically.
- Attempting to price above the contracted rate logs an override and triggers an approval rule.
- Agreement history view shows all agreements for an account with version snapshots.

## Files & Objects Touched
- `packages/db/prisma/schema.prisma` — `SalesAgreement`, `AgreementLineItem`
- `apps/web/app/admin/agreements/page.tsx` — agreement list
- `apps/web/app/admin/agreements/[id]/page.tsx` — agreement detail + line items
- `packages/pricing/src/agreement-applicator.ts` — auto-applies contracted prices at quote time
- `packages/api/src/routers/sales-agreement.ts`
- `packages/jobs/src/agreement-expiry.ts` — expiry detection and renewal trigger

## Methodology
1. Define schema: `SalesAgreement` (portal_id, hubspot_company_id, status, start_date, end_date, owner_user_id, version); `AgreementLineItem` (agreement_id, product_id, contracted_price, max_discount_pct, min_quantity).
2. Admin agreement UI: company search (autocomplete via HubSpot company sync), date pickers, status badge, line item table with product search and price inputs.
3. Implement `agreement-applicator.ts`: called at quote initialization; queries for an active agreement matching `hubspot_company_id`; for each product on the quote, if an `AgreementLineItem` exists, applies `contracted_price` as the `unit_price` and locks it (sets `price_locked = true` on the line item).
4. Price lock enforcement: in the pricing engine, if `line_item.price_locked = true`, skip discount waterfall stages that would increase the price above the contracted rate; flag any override attempt for approval.
5. Admin approval flow for new agreements: new agreements start in `pending_approval`; a designated approver reviews and activates.
6. Implement `agreement-expiry` Inngest job: daily scan for agreements where `end_date` is within 60 or 30 days; creates a notification and queues a renewal reminder (integrated with T045).
