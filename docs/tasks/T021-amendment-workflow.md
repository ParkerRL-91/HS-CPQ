# T021 — Mid-Term Amendment Workflow

**Sprint:** [Sprint 05](../sprints/sprint-05.md) | **Phase:** 3

## User Stories
- As a rep, I can initiate an amendment to add seats, change quantity, upgrade a product, or terminate a subscription mid-term.
- As a rep, the proration amount is automatically calculated so I don't need to do the math.
- As an auditor, every amendment generates a new quote linked to the original subscription creating a complete paper trail.

## Definition of Success
- Amendment types: Add-on/Upsell, Quantity Increase/Decrease, Upgrade/Edition Change, Early Termination.
- Proration is calculated based on days remaining in the contract term at the effective date.
- Each amendment creates a new `Quote` record (type = amendment) linked to the `Subscription`.
- `SubscriptionAmendment` record captures: type, effective_date, delta_quantity, proration_amount, linked quote_id.
- Early termination calculates any applicable early termination fee per configured policy.

## Files & Objects Touched
- `packages/subscriptions/src/amendment-calculator.ts` — proration and termination fee math
- `packages/subscriptions/src/amendment-service.ts` — orchestrates amendment quote creation and subscription update
- `packages/db/prisma/schema.prisma` — `SubscriptionAmendment`
- `apps/web/app/subscriptions/[id]/amend/page.tsx` — amendment initiation UI
- `packages/api/src/routers/subscription.ts` — `createAmendment` procedure

## Methodology
1. Implement `amendment-calculator.ts` proration: `proratedAmount = (dailyRate * remainingDays)` where `dailyRate = (annualContractValue / 365)`. For quantity increase: `(newQty - oldQty) * dailyRate * remainingDays`. For upgrade: credit = `old_product_daily_rate * remainingDays`; charge = `new_product_daily_rate * remainingDays`; net = charge - credit.
2. Implement early termination fee: configurable per product as a percentage of remaining contract value or a flat fee.
3. Build amendment initiation UI: select amendment type → type-specific form (quantity input for add-on/change; product selector for upgrade; confirmation for termination) → proration preview → submit.
4. `createAmendment` procedure: runs the calculator, creates the amendment quote, creates the `SubscriptionAmendment` record, updates `Subscription` fields (quantity, product, status) with the effective date.
5. If the amendment triggers approval thresholds, the amendment quote goes through the standard approval flow before the subscription is updated.
6. Test proration math: add-on on day 90 of a 365-day contract, upgrade with credit exceeding charge (should result in a credit memo, not a charge).
