# T023 — MRR/ARR Tracking

**Sprint:** [Sprint 05](../sprints/sprint-05.md) | **Phase:** 3

## User Stories
- As a manager, I can see accurate MRR and ARR per account that reflects amendments and cancellations, not just original deal amounts.
- As the system, MRR/ARR values are written back to HubSpot company properties so they appear in native HubSpot reports.
- As the platform, historical MRR data is tracked over time for trend analysis.

## Definition of Success
- MRR is calculated from all active `Subscription` records scoped to a company: `sum(monthly_equivalent(contracted_price * quantity))` for each active subscription.
- ARR = MRR × 12.
- MRR/ARR is recalculated on every subscription change (creation, amendment, cancellation).
- Values are written to HubSpot company custom properties `cpq_mrr` and `cpq_arr` within 60 seconds of a subscription change.
- A `MrrSnapshot` table records MRR per portal and per company on a daily basis for trend charts.

## Files & Objects Touched
- `packages/subscriptions/src/mrr-calculator.ts` — MRR computation from active subscriptions
- `packages/jobs/src/mrr-sync.ts` — Inngest job triggered on subscription changes
- `packages/jobs/src/mrr-snapshot.ts` — daily snapshot job
- `packages/hubspot/sync/company-sync.ts` — writes MRR/ARR to HubSpot company properties
- `packages/db/prisma/schema.prisma` — `MrrSnapshot` model

## Methodology
1. Implement `mrr-calculator.ts`: queries all active subscriptions for a given `portal_id` (and optionally `hubspot_company_id`); normalizes each subscription's `contracted_price * quantity` to a monthly value based on `billing_frequency` (monthly = 1x, quarterly = /3, annual = /12); sums them.
2. Define `mrr-sync` Inngest job: triggered by subscription create/update/delete events; calls `mrr-calculator` for the affected company; writes results to HubSpot company via `company-sync.ts`.
3. `company-sync.ts`: patches `cpq_mrr` and `cpq_arr` properties on the HubSpot company object via CRM API; ensures HubSpot custom properties are created on app install if they don't exist.
4. Define `mrr-snapshot` daily Inngest job: for each active portal, calculates MRR at the company and portal level; writes to `MrrSnapshot` table with a `snapshot_date`.
5. Register HubSpot custom properties for `cpq_mrr`, `cpq_arr`, `cpq_subscription_count`, `cpq_renewal_health_score` during the app install flow.
6. Test: company with mixed billing frequencies, company with cancelled subscriptions (excluded), company with amendments (uses amended quantity/price).
