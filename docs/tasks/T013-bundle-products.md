# T013 — Bundle Products

**Sprint:** [Sprint 03](../sprints/sprint-03.md) | **Phase:** 2

## User Stories
- As an admin, I can define a bundle product with required and optional components so that reps can sell packaged offerings.
- As a rep, when I add a bundle to a quote the components appear as child line items and the bundle price is calculated automatically.
- As a rep, I can add or remove optional components within a bundle without breaking the bundle's tracking.

## Definition of Success
- `Product` supports a `bundle_type` flag and a `BundleComponent` join table (product_id, component_product_id, is_optional, default_quantity).
- Bundle pricing methods supported: sum of components, fixed bundle price, percent discount off sum of components.
- When a bundle is added to a quote, required components are auto-added as child `LineItem` records with `parent_line_item_id` set.
- Reps can toggle optional components in the builder; the bundle price recalculates.
- Removing the parent bundle removes all child line items.

## Files & Objects Touched
- `packages/pricing/src/methods/bundle.ts` — bundle price calculation
- `packages/db/prisma/schema.prisma` — `BundleComponent` model, `LineItem.parent_line_item_id`
- `apps/web/components/quote-builder/BundleLineItem.tsx` — bundle row with collapsible component list
- `packages/api/src/routers/quote.ts` — `addBundle`, `toggleBundleComponent` procedures
- `apps/web/app/admin/products/[id]/bundle/page.tsx` — bundle component configurator

## Methodology
1. Extend `schema.prisma`: add `BundleComponent` table with `bundle_product_id`, `component_product_id`, `is_optional`, `default_quantity`, `sort_order`. Add `parent_line_item_id` to `LineItem`.
2. Implement `bundle.ts`: computes sum-of-components (run pricing engine on each component separately), fixed price (ignore component prices), or percent-off-sum (apply discount to component sum).
3. `addBundle` tRPC procedure: creates the parent line item, then iterates required components and creates child line items with `parent_line_item_id`.
4. `toggleBundleComponent` procedure: adds or removes an optional component child line item; triggers bundle price recalculation.
5. Build `BundleLineItem` component: parent row shows bundle name and total price; expanding shows component rows (required components greyed out/non-removable, optional components with a toggle). Each component shows its individual price.
6. Admin bundle configurator: product search to add components, required/optional toggle, default quantity input, drag-to-reorder.
