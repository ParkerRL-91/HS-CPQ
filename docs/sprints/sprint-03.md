# Sprint 03 — Advanced Pricing

**Phase:** 2 | **Weeks:** 9–12

## Goal
Implement enterprise-grade pricing capabilities: multi-dimensional pricing matrices, discount schedules with volume tiers and stacking policies, a data-driven price rules engine, and bundle product support. By the end of the sprint, the pricing engine can handle the full discount waterfall and any combination of pricing dimensions.

## Summary
This sprint closes the gap to Salesforce CPQ's pricing layer. Multi-dimensional pricing (MDQ) stores price cells indexed by two or more dimensions (quantity × term, quantity × tier). Discount schedules drive the volume-discount waterfall with configurable stacking. The price rules engine evaluates condition/action pairs to apply automatic adjustments, constraints, and alerts. Bundle products model parent-child relationships with flexible bundle pricing methods.

## Tasks
| ID | Task | Owner |
|----|------|-------|
| [T010](../tasks/T010-multi-dimensional-pricing.md) | Multi-Dimensional Pricing | |
| [T011](../tasks/T011-discount-schedules.md) | Discount Schedules | |
| [T012](../tasks/T012-price-rules-engine.md) | Price Rules Engine | |
| [T013](../tasks/T013-bundle-products.md) | Bundle Products | |

## Definition of Done
- Admin can define a pricing matrix with ≥2 dimensions; the engine looks up the correct cell at quote time.
- Volume discount schedules apply automatically based on quantity, with correct stacking behavior.
- Price rules fire in configured execution order; condition/action pairs are evaluated for all active rules.
- Bundle products price correctly using sum-of-components, fixed price, and percent-off-sum methods.
- All pricing changes are recorded in the pricing audit trail per quote version.
