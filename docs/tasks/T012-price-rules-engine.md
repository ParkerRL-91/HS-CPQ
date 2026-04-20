# T012 — Price Rules Engine

**Sprint:** [Sprint 03](../sprints/sprint-03.md) | **Phase:** 2

## User Stories
- As an admin, I can define rules that automatically adjust prices, enforce product constraints, show alerts, or configure quote attributes based on deal conditions.
- As a rep, when a rule fires the effect is applied automatically and I see a clear explanation of why the price changed.
- As a developer, rules are evaluated in a defined execution order with no side effects between them.

## Definition of Success
- Rule types supported: Price Adjustment, Product Constraint (require/exclude), Alert, Configuration (set quote fields), Quantity (enforce minimums).
- Conditions support: product presence, company property, quote attribute, discount threshold, contract term.
- Rules fire in `execution_order`; later rules can see the output of earlier rules.
- Each fired rule appends an entry to the quote's `PriceRuleApplication` audit log.
- Admin rule builder UI allows creating conditions (IF field OP value) and actions (THEN action type + params) without code.

## Files & Objects Touched
- `packages/pricing/src/rule-evaluator.ts` — condition evaluation and action dispatch
- `packages/pricing/src/rule-actions/` — one file per action type
- `packages/db/prisma/schema.prisma` — `PriceRule` model (`condition_set` JSON, `action` JSON, `execution_order`)
- `apps/web/app/admin/price-rules/page.tsx` — rule list
- `apps/web/components/admin/RuleBuilder.tsx` — drag-and-drop condition/action builder
- `packages/api/src/routers/price-rule.ts`

## Methodology
1. Define `PriceRule` schema: `portal_id`, `name`, `condition_set` (JSON: `{operator: AND|OR, conditions: [{field, op, value}]}`), `action` (JSON: `{type, params}`), `execution_order`, `active`.
2. Implement `rule-evaluator.ts`: loads all active rules for the portal, sorts by `execution_order`, evaluates each condition set against the current quote context. If conditions pass, dispatches to the appropriate action handler.
3. Action handlers: `price-adjustment` (apply % or $ change to target line items), `constraint` (add required product or flag incompatible product), `alert` (push a UI warning), `configuration` (set a quote field), `quantity` (enforce minimum quantity).
4. Build `RuleBuilder.tsx`: condition rows with field selector (dropdown of available context fields), operator selector, and value input. Action section with type selector and dynamic params form.
5. Rules are re-evaluated on every line item change and on quote recalculation; results cached in the quote's in-memory context for the duration of the request.
6. Write integration tests: rule with AND conditions, rule with OR conditions, two rules with ordering dependency.
