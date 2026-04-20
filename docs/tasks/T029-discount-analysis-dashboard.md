# T029 — Discount Analysis Dashboard

**Sprint:** [Sprint 06](../sprints/sprint-06.md) | **Phase:** 3

## User Stories
- As a manager, I can see average discounts by rep and product to identify where pricing discipline is weakest.
- As a VP, I can see a discount distribution histogram to understand the spread of discounting behavior across the team.
- As an admin, I can see how often floor prices are breached and whether approvals are resolving them correctly.

## Definition of Success
- Average discount by rep, product, product family, and customer segment — all filterable.
- Discount distribution histogram: X-axis = discount % buckets (0–5%, 5–10%, etc.), Y-axis = number of quotes.
- Floor price breach table: product, breach count, average breach magnitude, resolution rate.
- Revenue at risk metric: sum of `quote.total` for quotes in pending_approval status with manual discounts.
- All metrics pull from `DiscountWaterfallLog` for accuracy.

## Files & Objects Touched
- `apps/web/app/dashboard/discounts/page.tsx`
- `apps/web/components/dashboard/DiscountByRepTable.tsx`
- `apps/web/components/dashboard/DiscountHistogram.tsx`
- `apps/web/components/dashboard/FloorPriceBreachTable.tsx`
- `packages/api/src/routers/dashboard.ts` — `getDiscountMetrics` procedure
- `packages/db/src/queries/discounts.ts`

## Methodology
1. Average discount query: join `Quote` → `LineItem`; average `discount_pct` grouped by `owner_user_id` / `product_id` / `product_family` / `company_segment`; apply date range and portal_id filters.
2. Histogram query: bucket `quote.discount_pct` into 5% ranges using `FLOOR(discount_pct / 5) * 5` and count per bucket.
3. Floor price breach query: scan `DiscountWaterfallLog` for records where a floor price cap was applied (flagged by the pricing engine); group by product, count occurrences, calculate average breach magnitude `(floor_price - pre_cap_price) / floor_price * 100`.
4. Revenue at risk: sum `quote.base_currency_total` for quotes where `status = pending_approval` and at least one `LineItem.discount_pct > threshold`.
5. Build `DiscountHistogram` using a bar chart; render `DiscountByRepTable` as a sortable table with conditional formatting (high discount % rows in amber/red).
6. All discount analysis queries use `DiscountWaterfallLog` as the source of truth, not the raw `LineItem.discount_pct`, to capture the full waterfall breakdown.
