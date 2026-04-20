# Sprint 09 — Usage-Based Pricing & Agreement Expiry

**Phase:** 5 | **Weeks:** 33–37

## Goal
Implement all usage-based and metered pricing models (pay-per-use, overage, prepaid credits, tiered usage, hybrid) with committed minimums and estimated usage on line items. Automate Sales Agreement expiry workflows with notification sequences and auto-generated renewal agreements.

## Summary
Usage-based pricing is increasingly standard in SaaS. This sprint extends the pricing engine with five new pricing models and the corresponding data model fields. The CPQ's role is to define the structure at quote time; actual metering is handled downstream — but the quote must clearly communicate the pricing model, estimate ranges, and overage structure. Agreement expiry workflows ensure contracted pricing relationships renew on time rather than silently lapsing.

## Tasks
| ID | Task | Owner |
|----|------|-------|
| [T044](../tasks/T044-usage-based-pricing-engine.md) | Usage-Based & Metered Pricing Engine | |
| [T045](../tasks/T045-sales-agreement-expiry-workflows.md) | Sales Agreement Expiry Workflows | |

## Definition of Done
- All five usage pricing models are selectable per product and render correctly on quote output with estimated cost ranges.
- `committed_minimum`, `estimated_usage`, and `usage_rate_snapshot` are stored per line item.
- Renewal quotes carry forward usage pricing structure from the originating subscription.
- Sales Agreements notify account owners at 60 and 30 days before expiry.
- A renewal agreement is auto-generated at the 30-day trigger with current terms as baseline.
