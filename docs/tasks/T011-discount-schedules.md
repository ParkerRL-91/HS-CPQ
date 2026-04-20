# T011 — Discount Schedules

**Sprint:** [Sprint 03](../sprints/sprint-03.md) | **Phase:** 2

## User Stories
- As an admin, I can define volume discount schedules with quantity tiers and discount percentages so that large orders are automatically discounted.
- As a rep, when I add a product to a quote the applicable discount tier is automatically applied based on the quantity.
- As a rep, I can see the full discount waterfall — showing each stage's contribution — so that I can explain pricing to the prospect.

## Definition of Success
- `DiscountSchedule` table stores tiers (min qty, max qty, discount %) and stacking policy (additive, compound, override).
- Schedules are assignable to individual products or product families.
- The pricing engine applies all applicable schedules in the correct waterfall order (partner → promotional → volume → customer tier → manual → distributor).
- Each stage records its applied discount in the `calculationLog`.
- A floor price check runs after the full waterfall; if the result is below `product.floor_price`, the discount is capped.

## Files & Objects Touched
- `packages/pricing/src/discount-waterfall.ts` — orchestrates the six-stage discount waterfall
- `packages/pricing/src/schedules/volume.ts` — volume schedule tier lookup
- `packages/db/prisma/schema.prisma` — `DiscountSchedule`, `DiscountTier` models
- `apps/web/app/admin/discount-schedules/page.tsx` — schedule list and builder UI
- `apps/web/components/admin/DiscountTierTable.tsx`
- `packages/api/src/routers/discount-schedule.ts` — tRPC CRUD

## Methodology
1. Define `DiscountSchedule` schema: `portal_id`, `name`, `type` (volume | customer_tier | partner), `stacking_policy` (additive | compound | override), `tiers` (JSON: `{min_qty, max_qty, discount_pct}[]`), `max_discount_cap`, `best_price_guarantee` (bool).
2. Implement `volume.ts`: given quantity and a schedule's tiers, returns the applicable tier's `discount_pct`.
3. Implement `discount-waterfall.ts`: sequentially applies each discount stage. Stacking: additive = sum of percentages off list; compound = each stage applies to the running price; override = last stage wins.
4. After waterfall, check `runningPrice < product.floor_price`; if true, cap discount so price equals `floor_price` and log a warning.
5. Build admin schedule builder: tier table with add/remove rows, stacking policy selector, cap and floor inputs.
6. Assign schedules to products via a multi-select in the product admin UI.
