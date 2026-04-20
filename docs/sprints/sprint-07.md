# Sprint 07 — Template Engine & Guided Selling

**Phase:** 4 | **Weeks:** 25–28

## Goal
Ship the dynamic template engine with a WYSIWYG editor, conditional section logic, and token insertion. Implement the guided selling wizard with answer branching and auto-add. Build the upsell/cross-sell recommendation engine with rule-based and historical-pattern recommendations.

## Summary
Templates transform quotes from data dumps into professional, personalized proposals. This sprint builds the editor (drag-and-drop sections, token picker, conditional logic builder, live preview) and the rendering engine that evaluates conditions and substitutes tokens at generation time. Guided selling structures the product selection experience so reps confidently recommend the right products. The recommendation engine surfaces upsell and cross-sell opportunities without interrupting the quoting flow.

## Tasks
| ID | Task | Owner |
|----|------|-------|
| [T032](../tasks/T032-template-engine.md) | WYSIWYG Template Engine | |
| [T033](../tasks/T033-conditional-logic.md) | Conditional Section Logic | |
| [T034](../tasks/T034-guided-selling-wizard.md) | Guided Selling Wizard | |
| [T035](../tasks/T035-upsell-crosssell-engine.md) | Upsell & Cross-Sell Recommendation Engine | |

## Definition of Done
- Admins can build, preview, and version-control quote templates in the WYSIWYG editor.
- Dynamic tokens resolve correctly from deal, company, contact, and rep data.
- Conditional sections show/hide correctly based on configured conditions at PDF generation time.
- Guided selling wizard filters product catalog based on answers; high-confidence products auto-add to quote.
- Recommendation panel shows upsell/cross-sell suggestions with one-click add and labeled reason.
