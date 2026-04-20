# T005 — Quote Builder UI

**Sprint:** [Sprint 02](../sprints/sprint-02.md) | **Phase:** 1

## User Stories
- As a rep, I can search and add products to a quote so that I can configure the deal's line items.
- As a rep, I can set quantities and apply a manual discount per line item so that I can tailor pricing to the deal.
- As a rep, I can reorder line items via drag-and-drop so that the quote reads in the order I want.
- As a rep, I see a running total that updates instantly as I make changes so that I can track the deal value.

## Definition of Success
- Quote builder is a full-page Next.js route (`/quotes/[quoteId]/build`) launched from the sidebar.
- Product search filters the synced product catalog with debounced input; results render within 200ms.
- Adding a product creates a `LineItem` record and immediately calls the pricing engine; unit price and extended price display.
- Manual discount (percent or fixed amount) is enterable per line item; total recalculates.
- Line items can be reordered via drag-and-drop using dnd-kit; order is persisted.
- Quote summary footer shows subtotal, total discount, and grand total in real time.

## Files & Objects Touched
- `apps/web/app/quotes/[quoteId]/build/page.tsx` — route
- `apps/web/components/quote-builder/ProductSearch.tsx`
- `apps/web/components/quote-builder/LineItemRow.tsx`
- `apps/web/components/quote-builder/QuoteSummary.tsx`
- `packages/api/src/routers/quote.ts` — tRPC procedures: `addLineItem`, `updateLineItem`, `removeLineItem`, `reorderLineItems`
- `packages/db/prisma/schema.prisma` — `Quote`, `LineItem`

## Methodology
1. Create the `/quotes/[quoteId]/build` Next.js page; load quote and line items via tRPC `quote.get`.
2. Build `ProductSearch` component: controlled input with 300ms debounce triggers tRPC `product.search`; results displayed as a card list with "Add" button.
3. On add, call tRPC `quote.addLineItem` which creates the `LineItem` and runs the pricing engine; returns enriched line item with computed prices.
4. Build `LineItemRow` with inline quantity input, discount input (toggle % or $), and delete button; each change triggers `quote.updateLineItem` + repricing.
5. Integrate dnd-kit `DndContext` + `SortableContext` around line items; on drag end, call `quote.reorderLineItems` with the new order array.
6. `QuoteSummary` component subscribes to local quote state (Zustand) and derives totals client-side for instant feedback; persisted totals are recalculated server-side on save.
