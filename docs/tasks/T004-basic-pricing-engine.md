# T004 — Basic Pricing Engine

**Sprint:** [Sprint 01](../sprints/sprint-01.md) | **Phase:** 1

## User Stories
- As a rep, when I add a product to a quote the correct price is automatically calculated based on the product's pricing method and quantity.
- As a developer, I can write unit tests for every pricing method without a database or API dependency.
- As the platform, pricing logic is isolated from the UI layer so it can be extracted as a microservice later.

## Definition of Success
- Engine is a pure TypeScript module with no database calls, no HTTP calls, and no side effects.
- Supports: List Price, Per Unit, Flat Fee, Block Pricing, Percent of Total.
- Unit test coverage ≥ 90% for all pricing method branches, including edge cases (quantity = 0, quantity at block boundary, percent target not yet in line items).
- Engine accepts a typed `PricingInput` and returns a typed `PricingResult` with `unit_price`, `extended_price`, `applied_method`, and `calculation_log`.

## Files & Objects Touched
- `packages/pricing/src/engine.ts` — main pricing engine entry point
- `packages/pricing/src/methods/list-price.ts`
- `packages/pricing/src/methods/per-unit.ts`
- `packages/pricing/src/methods/flat-fee.ts`
- `packages/pricing/src/methods/block-pricing.ts`
- `packages/pricing/src/methods/percent-of-total.ts`
- `packages/pricing/src/types.ts` — `PricingInput`, `PricingResult`, `PricingMethod` enum
- `packages/pricing/src/__tests__/` — unit tests per method

## Methodology
1. Define `PricingInput` type: `{ product, quantity, quoteLineItems, pricingMethod, listPrice, blockSchedule?, percentTarget? }`.
2. Define `PricingResult` type: `{ unitPrice, extendedPrice, appliedMethod, calculationLog: string[] }`.
3. Implement each method as a pure function: receives input, returns result, records each calculation step in `calculationLog` for transparency.
4. `block-pricing.ts`: iterates block schedule, finds the block containing `quantity`, applies that unit price (all-units model — all units priced at the matched block rate).
5. `percent-of-total.ts`: sums the `extendedPrice` of target line items already in the quote; computes the percentage.
6. `engine.ts`: dispatches to the correct method based on `pricingMethod`; wraps errors with context.
7. Write unit tests for each method covering nominal cases, boundary values, and invalid inputs (expect typed errors, not thrown exceptions).
