# T044 — Usage-Based & Metered Pricing Engine

**Sprint:** [Sprint 09](../sprints/sprint-09.md) | **Phase:** 5

## User Stories
- As an admin, I can configure a product with a usage-based pricing model (pay-per-use, overage, prepaid credits, tiered usage, or hybrid).
- As a rep, when I add a usage-based product to a quote I can set the committed minimum and enter an estimated usage so the prospect sees a realistic cost range.
- As a prospect, the quote clearly communicates the pricing model, what's included, and what the overage rate is.

## Definition of Success
- All five usage pricing models are configurable per product: pay-per-use, overage, prepaid credits, tiered usage, hybrid.
- Line items for usage-based products expose `committed_minimum`, `estimated_usage`, and display an estimated cost range (min = committed minimum cost, max = estimated usage cost).
- `usage_rate_snapshot` is stored on the line item at quote time for audit and renewal.
- Renewal engine (T022) carries forward the usage pricing structure from the subscription.
- Quote PDF and hosted page render usage pricing clearly: base price, included allotment, overage rate, estimated cost.

## Files & Objects Touched
- `packages/pricing/src/methods/usage-based.ts` — usage pricing calculation
- `packages/db/prisma/schema.prisma` — `Product.usage_model`, `Product.usage_tiers`, `Product.included_allotment`, `LineItem.committed_minimum`, `LineItem.estimated_usage`, `LineItem.usage_rate_snapshot`
- `apps/web/components/quote-builder/UsageLineItemConfig.tsx` — usage product configuration panel
- `packages/pdf/src/templates/` — usage pricing section template
- `packages/api/src/routers/product.ts` — usage config endpoints
- `apps/web/app/admin/products/[id]/pricing/page.tsx` — usage pricing admin UI

## Methodology
1. Extend `Product` schema: `usage_model` enum (`pay_per_use | overage | prepaid_credits | tiered_usage | hybrid`), `usage_unit_label`, `usage_tiers` (JSON: `{min, max, rate}[]`), `included_allotment`, `credit_block_size` (for prepaid credits).
2. Extend `LineItem` schema: `committed_minimum`, `estimated_usage`, `usage_rate_snapshot` (JSON snapshot of the product's usage config at quote time).
3. Implement `usage-based.ts` calculation for each model:
   - Pay-per-use: `estimated_cost = estimated_usage * rate`
   - Overage: `estimated_cost = base_price + max(0, estimated_usage - included_allotment) * overage_rate`
   - Prepaid credits: `line_price = ceil(committed_minimum / credit_block_size) * credit_block_price`
   - Tiered usage: sum each tier's contribution (tiered accumulation, not block)
   - Hybrid: `base_price + tiered_usage_cost`
4. Build `UsageLineItemConfig` panel: shown when a usage-based product is on the quote; inputs for committed minimum and estimated usage; renders real-time min/max cost range.
5. Snapshot usage config to `usage_rate_snapshot` on line item save so the pricing is auditable even if product config changes.
6. Update PDF template: add a "Usage Pricing" section that renders the model type, included allotment, usage tiers table (if applicable), overage rate, and the min/max estimated cost range.
