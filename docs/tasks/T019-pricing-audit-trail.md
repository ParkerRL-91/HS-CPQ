# T019 — Pricing Audit Trail

**Sprint:** [Sprint 04](../sprints/sprint-04.md) | **Phase:** 2

## User Stories
- As a manager, I can see exactly which pricing rules, discount stages, and manual overrides contributed to the final price on any quote.
- As an auditor, the pricing audit log is immutable — it cannot be edited after the fact.
- As a rep, I can show the prospect a clear breakdown of how the price was derived.

## Definition of Success
- Every pricing calculation generates a `PriceRuleApplication` record per rule fired, linked to the quote version.
- Discount waterfall stages are recorded per line item: list price → partner discount → promo → volume → tier → manual → distributor → final price.
- Audit records are written in an append-only pattern; no update or delete operations are permitted on audit tables.
- Admin and manager roles can view the audit trail via a dedicated panel on the quote detail page.
- The audit trail is included in the quote's version snapshot.

## Files & Objects Touched
- `packages/db/prisma/schema.prisma` — `PriceRuleApplication`, `DiscountWaterfallLog` models
- `packages/pricing/src/audit-logger.ts` — helper that writes immutable audit records
- `apps/web/components/quotes/PricingAuditPanel.tsx` — read-only audit trail display
- `packages/api/src/routers/quote.ts` — audit log query procedure

## Methodology
1. Define `PriceRuleApplication`: `quote_version_id`, `line_item_id`, `rule_id`, `rule_name`, `applied_at`, `action_type`, `effect_description` (read-only fields; no update endpoint).
2. Define `DiscountWaterfallLog`: `quote_version_id`, `line_item_id`, `stage` (enum: list|partner|promo|volume|tier|manual|distributor), `discount_pct`, `running_price`, `applied_at`.
3. Implement `audit-logger.ts`: exposes `logRuleApplication(...)` and `logDiscountStage(...)` pure functions that write to the database using `prisma.$executeRawUnsafe` or a dedicated write-only Prisma client with no `update`/`delete` methods exposed.
4. Call audit logger from the pricing engine and rule evaluator at each stage; pass through the active `quote_version_id`.
5. Build `PricingAuditPanel` component: tabbed view showing "Discount Waterfall" (table of stages per line item) and "Rules Applied" (list of fired rules with their effect). Accessible to manager and admin roles only.
6. Include audit log data in the quote version snapshot (T025) so historical audits are preserved even if rules change.
