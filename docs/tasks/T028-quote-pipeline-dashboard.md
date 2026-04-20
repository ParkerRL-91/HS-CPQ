# T028 — Quote Pipeline Dashboard

**Sprint:** [Sprint 06](../sprints/sprint-06.md) | **Phase:** 3

## User Stories
- As a manager, I can see all active quotes organized by stage so I understand the health of the quote pipeline.
- As a manager, I can see which quotes are expiring soon so I can prompt reps to follow up.
- As a rep, I can see my own quote performance metrics (win rate, average time to close) to understand how I'm trending.

## Definition of Success
- Dashboard renders in < 2 seconds with real data.
- Stage breakdown shows count and total value per quote stage (Draft, Pending Approval, Approved, Sent, Accepted, Expired).
- Win/loss table filterable by rep, product family, and date range.
- "Expiring soon" widget shows quotes expiring in 7, 14, and 30 days with one-click navigation to the quote.
- All charts are filterable by date range, rep, and product family.

## Files & Objects Touched
- `apps/web/app/dashboard/pipeline/page.tsx`
- `apps/web/components/dashboard/StageBreakdown.tsx`
- `apps/web/components/dashboard/WinLossTable.tsx`
- `apps/web/components/dashboard/ExpiringQuotesWidget.tsx`
- `packages/api/src/routers/dashboard.ts` — `getPipelineMetrics` procedure
- `packages/db/src/queries/pipeline.ts` — aggregate SQL queries

## Methodology
1. Design the data queries in `pipeline.ts`: group quotes by `status`, sum `total` per group, and compute count. Filter by `portal_id`, optional `owner_user_id`, optional product family (via line item join), and date range on `created_at`.
2. Win rate query: count `status = accepted` divided by count `status in (accepted, expired)` within the period; group by `owner_user_id` and `product_family`.
3. Average time-to-close: average of `accepted_at - created_at` for accepted quotes; group by rep and product family.
4. Build `StageBreakdown` as a horizontal bar chart (use Recharts or Tremor); bars colored by stage status.
5. `ExpiringQuotesWidget`: sorted table of quotes where `valid_until` is within 30 days and `status = sent`; columns: company, value, rep, days remaining; row click navigates to quote.
6. Implement dashboard filter bar (date range picker, rep multi-select, product family multi-select) as a shared URL-state-driven component that all dashboard pages can use.
