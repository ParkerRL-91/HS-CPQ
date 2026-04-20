# Sprint 04 — Approvals, Multi-Currency & Sales Agreements

**Phase:** 2 | **Weeks:** 13–16

## Goal
Implement the full approval system (sequential + parallel chains, email/Slack notifications, audit log), multi-currency support with configurable exchange rate modes, and the Sales Agreements module for contracted pricing. Complete Phase 2 with sidebar enhancements and HubSpot Workflow Actions registered.

## Summary
Approval chains are the compliance layer between discounting and sending. The state machine tracks every approval action and records an immutable audit log. Multi-currency lets sales teams quote in any configured currency, with all approval thresholds enforced in the portal's base currency. Sales Agreements lock contracted prices to specific accounts, auto-applying them when a matching deal creates a quote — preventing accidental over-quoting.

## Tasks
| ID | Task | Owner |
|----|------|-------|
| [T014](../tasks/T014-approval-engine.md) | Approval Engine (State Machine) | |
| [T015](../tasks/T015-approval-ui.md) | Approval UI (Approver Review Screen) | |
| [T016](../tasks/T016-approval-admin.md) | Approval Admin (Rule Builder) | |
| [T017](../tasks/T017-multi-currency.md) | Multi-Currency Support | |
| [T018](../tasks/T018-sales-agreements.md) | Sales Agreements | |
| [T019](../tasks/T019-pricing-audit-trail.md) | Pricing Audit Trail | |

## Definition of Done
- Sequential and parallel approval chains route correctly; first-response and delegated modes work.
- Approvers receive email (and optionally Slack) notifications with deep links.
- All approval actions are recorded with approver identity, timestamp, action, and quote snapshot.
- Quotes can be created in any configured currency; reporting currency conversion is applied.
- Active Sales Agreements auto-apply contracted prices to matching quote line items.
- All Phase 2 HubSpot Workflow Actions are registered and callable from HubSpot automation.
