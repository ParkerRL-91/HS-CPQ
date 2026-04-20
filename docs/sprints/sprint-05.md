# Sprint 05 — Subscriptions & Renewals

**Phase:** 3 | **Weeks:** 17–20

## Goal
Build the complete subscription lifecycle: auto-create subscription records on closed-won deals, support mid-term amendments with proration, automatically generate renewal quotes at configurable lead times, and surface MRR/ARR data back in HubSpot.

## Summary
Recurring revenue products need a subscription record that outlives the originating quote. This sprint creates those records, handles every mid-term change type (add-on, quantity change, upgrade, early termination) with correct proration math, and runs daily renewal reminder jobs that auto-generate renewal quotes. MRR/ARR is calculated from live subscription data and written back to HubSpot company properties so it appears in native HubSpot reports.

## Tasks
| ID | Task | Owner |
|----|------|-------|
| [T020](../tasks/T020-subscription-records.md) | Subscription Records | |
| [T021](../tasks/T021-amendment-workflow.md) | Mid-Term Amendment Workflow | |
| [T022](../tasks/T022-renewal-engine.md) | Renewal Engine | |
| [T023](../tasks/T023-mrr-arr-tracking.md) | MRR/ARR Tracking | |
| [T024](../tasks/T024-renewal-notifications.md) | Renewal Notifications | |

## Definition of Done
- Subscription records are created automatically when a deal with recurring line items moves to Closed Won.
- All four amendment types (add-on, quantity change, upgrade, early termination) calculate proration correctly.
- Renewal quotes are auto-generated at 90/60/30-day windows with the correct renewal pricing method applied.
- MRR and ARR values are calculated from subscription records and written to HubSpot company properties.
- Reps and managers receive renewal reminder notifications at each configured window.
