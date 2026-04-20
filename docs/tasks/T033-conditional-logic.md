# T033 — Conditional Section Logic

**Sprint:** [Sprint 07](../sprints/sprint-07.md) | **Phase:** 4

## User Stories
- As an admin, I can configure a template section to only appear if a certain condition is true (e.g., show the HIPAA addendum only if company.industry = Healthcare).
- As a rep, when the PDF is generated, sections that don't apply to this deal are automatically excluded.
- As an admin, I can test conditional logic in preview mode by toggling deal attributes.

## Definition of Success
- Any section in the template editor can have a condition attached (IF field OP value).
- Supported condition fields: product presence, deal size, contract term, company type, company country, quote discount %, custom HubSpot deal/company properties.
- At render time, conditions are evaluated against the deal context; sections evaluate to shown or hidden.
- Preview mode shows a "condition result" indicator on each conditional section so admins can verify logic.
- Nested conditions (AND/OR groups) are supported.

## Files & Objects Touched
- `apps/web/components/admin/SectionConditionBuilder.tsx` — condition editor for a template section
- `packages/templates/src/condition-evaluator.ts` — evaluates conditions at render time
- `packages/templates/src/renderer.ts` — calls evaluator during template walk
- `apps/web/components/admin/TemplateEditor.tsx` — integrates condition builder into section UI

## Methodology
1. Extend the TipTap `Section` node to carry a `conditions` attribute: JSON array of `{field, operator, value}` with an `AND|OR` combinator.
2. Build `SectionConditionBuilder` component: triggered by a "Conditions" button on each section; opens a modal with condition rows (field selector, operator dropdown, value input); supports add/remove rows and AND/OR toggle.
3. Implement `condition-evaluator.ts`: receives the conditions array and the deal context object; evaluates each condition using the operator (`equals`, `not_equals`, `greater_than`, `contains`, `in`); combines results using the configured AND/OR logic; returns boolean.
4. In `renderer.ts`, before rendering each section's content, call `condition-evaluator.ts`; if the result is false, skip the section entirely (exclude from HTML output).
5. Preview mode enhancement: overlay each conditional section with a badge ("SHOWN" in green / "HIDDEN" in red) based on the evaluation result; clicking the badge shows which conditions passed/failed.
6. Write evaluator unit tests: all operator types, AND/OR combinations, missing field handling (treat as false), case-insensitive string comparison.
