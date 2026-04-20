# Sprint 02 — Core Quoting

**Phase:** 1 | **Weeks:** 5–8

## Goal
Reps can create a quote from a HubSpot deal, configure line items with pricing, generate a PDF, and deliver it to a prospect — all without leaving HubSpot. At the end of this sprint the full Phase 1 loop is closed: deal → quote → PDF → delivery → HubSpot sync.

## Summary
With the foundation in place, this sprint layers on the user-facing quoting experience. The Quote Builder UI lets reps add products, set quantities, and apply manual discounts. Puppeteer renders that data into a professional PDF. Resend delivers it via email or a hosted link. The HubSpot sidebar gives reps quick access from the Deal record. Object sync writes the final quote state back to HubSpot so CRM reporting stays accurate.

## Tasks
| ID | Task | Owner |
|----|------|-------|
| [T005](../tasks/T005-quote-builder-ui.md) | Quote Builder UI | |
| [T006](../tasks/T006-pdf-generation.md) | PDF Generation | |
| [T007](../tasks/T007-quote-delivery.md) | Quote Delivery (Hosted Link + Email) | |
| [T008](../tasks/T008-hubspot-sidebar-basic.md) | HubSpot Sidebar (Basic) | |
| [T009](../tasks/T009-hubspot-object-sync.md) | HubSpot Object Sync | |

## Definition of Done
- Rep can launch the CPQ configurator from the HubSpot Deal sidebar.
- Quote builder supports product selection, quantity editing, and manual discount entry.
- A pixel-accurate PDF is generated server-side with dynamic tokens.
- Quote is sent via hosted link and email attachment.
- Quote status, total, discount %, and line items are written back to HubSpot Deal and Quote records.
