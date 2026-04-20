# T030 — Renewal Health Dashboard

**Sprint:** [Sprint 06](../sprints/sprint-06.md) | **Phase:** 3

## User Stories
- As a manager, I can see all subscriptions by status and upcoming renewal dates so I can manage renewal risk.
- As a VP, I can see Net Revenue Retention (NRR) trend to understand how the installed base is growing or shrinking.
- As a manager, I can see the ARR waterfall showing new business, expansion, contraction, and churn.

## Definition of Success
- Subscription status breakdown: Active, In Renewal (renewal quote sent), At Risk (flagged), Churned.
- Upcoming renewals calendar view: 30/60/90-day windows with ARR value per window.
- Rolling 12-month NRR trend line chart.
- ARR waterfall chart: beginning ARR + new + expansion - contraction - churn = ending ARR, by month.
- Average renewal lead time: average days between renewal quote creation and acceptance.

## Files & Objects Touched
- `apps/web/app/dashboard/renewals/page.tsx`
- `apps/web/components/dashboard/SubscriptionStatusBreakdown.tsx`
- `apps/web/components/dashboard/NRRTrendChart.tsx`
- `apps/web/components/dashboard/ArrWaterfallChart.tsx`
- `apps/web/components/dashboard/UpcomingRenewalsCalendar.tsx`
- `packages/db/src/queries/renewals.ts`

## Methodology
1. Subscription status query: count and ARR sum per status, filtered by portal_id and date range. "At Risk" = active subscriptions where `end_date < 90 days` and `renewal_quote_id IS NULL`.
2. NRR formula: `(Beginning ARR + Expansion ARR - Contraction ARR - Churn ARR) / Beginning ARR` rolling over 12 months. Use `MrrSnapshot` table for historical ARR values; compute expansion/contraction from `SubscriptionAmendment` records.
3. ARR waterfall: for each month, query new subscriptions (new ARR), amendment records (expansion or contraction delta), and cancelled subscriptions (churn ARR). Build a stacked bar/waterfall chart.
4. Upcoming renewals calendar: group subscriptions by `end_date` bucket (30/60/90 days); show total count and ARR per bucket with clickable drill-down to individual subscriptions.
5. Average renewal lead time: `AVG(renewal_quote.created_at - subscription.end_date)` for subscriptions with accepted renewal quotes in the period.
6. All ARR values displayed in portal base currency, converted using the FX rate at the snapshot date.
