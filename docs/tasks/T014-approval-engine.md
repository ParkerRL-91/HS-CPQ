# T014 — Approval Engine (State Machine)

**Sprint:** [Sprint 04](../sprints/sprint-04.md) | **Phase:** 2

## User Stories
- As the system, when a quote triggers an approval rule an approval request is automatically created and routed to the correct approvers.
- As an approver, I can approve or reject a quote and my action is recorded with a comment and timestamp.
- As the system, the quote is locked from editing while awaiting approval so that the approved version is the version that gets sent.

## Definition of Success
- Approval triggers evaluate correctly against configured rules at quote submission time.
- Sequential chains: each step activates only after the prior step's approval.
- Parallel chains: all approvers notified simultaneously; all must approve for the request to resolve.
- First-response mode: first approver to act resolves the request.
- Delegated approval: if an approver has an active delegation rule, the backup approver is notified instead.
- Quote is locked (`status = pending_approval`) during the approval cycle; unlock on approval or rejection.
- Full state machine covers all transitions: Draft → Pending → In Review → Approved | Rejected | Recalled | Expired.

## Files & Objects Touched
- `packages/approvals/src/state-machine.ts` — quote approval state transitions
- `packages/approvals/src/trigger-evaluator.ts` — evaluates quote against approval trigger rules
- `packages/approvals/src/chain-executor.ts` — orchestrates sequential/parallel step progression
- `packages/db/prisma/schema.prisma` — `ApprovalRequest`, `ApprovalStep`, `ApprovalChain`
- `packages/jobs/src/approval-sla-check.ts` — Inngest job for SLA expiry
- `packages/api/src/routers/approval.ts` — `submit`, `approve`, `reject`, `recall` procedures

## Methodology
1. Implement `trigger-evaluator.ts`: loads all active `ApprovalChain` trigger rules for the portal; evaluates the quote's discount %, value, products, and terms against each rule's conditions. Returns a list of triggered chains.
2. Implement `state-machine.ts`: exports transition functions (`toSubmitted`, `toApproved`, `toRejected`, `toRecalled`, `toExpired`) that validate the current state before transitioning and record the new state with timestamp.
3. Implement `chain-executor.ts`: on chain activation, creates `ApprovalStep` records for the chain's configured approvers; for sequential chains, sets `step.order` and activates only the first; for parallel, activates all simultaneously.
4. `approve` procedure: marks the `ApprovalStep` as approved; if sequential and more steps remain, activates the next step; if all steps are approved, transitions quote to `approved`.
5. Define SLA check Inngest job: runs every hour, finds `pending_approval` quotes past their SLA deadline, auto-escalates (activates manager's step) or expires.
6. Write state machine tests covering every valid and invalid transition.
