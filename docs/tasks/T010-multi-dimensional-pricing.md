# T010 — Multi-Dimensional Pricing

**Sprint:** [Sprint 03](../sprints/sprint-03.md) | **Phase:** 2

## User Stories
- As an admin, I can define a pricing matrix with two or more dimensions (e.g., quantity tier × contract term) so that prices vary accurately across both axes.
- As a rep, when I select a product with a pricing matrix, the correct cell price is looked up automatically based on the quantity I enter and the contract term on the quote.
- As a developer, the matrix lookup is deterministic and fully unit-tested.

## Definition of Success
- Admin UI allows creating dimensions (name, values) and a matrix grid where each cell holds a unit price.
- `PricingMatrix` table stores dimension definitions and price cells as JSON.
- Pricing engine `mdq-lookup.ts` accepts dimension values, finds the correct cell, and returns the unit price.
- If no exact cell matches, the engine applies the nearest applicable row (floor lookup) with a warning in `calculationLog`.
- Unit tests cover: exact match, boundary match, no match (fallback), and 3-dimensional matrix.

## Files & Objects Touched
- `packages/pricing/src/methods/mdq-lookup.ts` — matrix cell lookup logic
- `packages/pricing/src/types.ts` — `PricingMatrix`, `MatrixCell`, `DimensionConfig` types
- `packages/db/prisma/schema.prisma` — `PricingMatrix` model
- `apps/web/app/admin/products/[id]/matrix/page.tsx` — matrix builder UI
- `apps/web/components/admin/MatrixGrid.tsx` — grid editor component
- `packages/api/src/routers/pricing-matrix.ts` — tRPC CRUD for matrices

## Methodology
1. Extend `schema.prisma`: `PricingMatrix` has `product_id`, `dimensions` (JSON array of `{name, values[]}`), `cells` (JSON array of `{dimension_values: Record<string, string>, price}`).
2. Build admin matrix editor: renders a 2D grid from the dimension definitions; each cell is an inline price input. For 3+ dimensions, renders a tabbed interface (one tab per value of the third dimension).
3. Implement `mdq-lookup.ts`: given a product's `PricingMatrix` and the quote's dimension values, iterates cells to find an exact match; if none, falls back to the closest match using a "floor" strategy per dimension.
4. Integrate MDQ lookup into the main pricing engine dispatcher: if `product.pricing_method = mdq`, call `mdq-lookup` to get the unit price before applying discounts.
5. Write unit tests for lookup edge cases: partial match, out-of-range values, missing dimension.
