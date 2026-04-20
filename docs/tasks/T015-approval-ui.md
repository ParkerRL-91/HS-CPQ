# T015 — Approval UI (Approver Review Screen)

**Sprint:** [Sprint 04](../sprints/sprint-04.md) | **Phase:** 2

## User Stories
- As an approver, I receive an email with a deep link to a review screen where I can see the full quote and make my approval decision.
- As an approver, I can see exactly what triggered the approval request (which rule) and the full quote including line items, pricing, and discounts.
- As a rep, I receive real-time status updates as each approver acts on my quote.

## Definition of Success
- Approval review screen (`/approvals/[requestId]`) is accessible only to the assigned approver for that request.
- Screen shows: trigger summary, quote line items and totals, discount waterfall, pricing audit trail, and any prior approver comments.
- Approver can Approve or Reject with a required comment on rejection.
- Rep receives email notification within 30 seconds of each approver action.
- Sidebar `ApprovalStatusBadge` updates in real time (polling or SSE) as approvers act.

## Files & Objects Touched
- `apps/web/app/approvals/[requestId]/page.tsx` — approval review page
- `apps/web/components/approvals/QuoteReviewPanel.tsx` — line items + pricing summary
- `apps/web/components/approvals/DiscountWaterfallTable.tsx`
- `apps/web/components/approvals/ApprovalActionForm.tsx` — approve/reject form with comment
- `packages/email/src/templates/approval-request.tsx` — email template for approvers
- `packages/email/src/templates/approval-status-update.tsx` — email template for reps
- `packages/api/src/routers/approval.ts` — `approve`, `reject` procedures

## Methodology
1. Build the review page: server-side load `ApprovalRequest` by ID, verify session user is the active approver for this step (else redirect to unauthorized).
2. Render `QuoteReviewPanel`: full line item table with quantity, list price, discounts applied at each waterfall stage, and final unit/extended prices. Include quote totals and the specific trigger reason.
3. Build `ApprovalActionForm`: radio group (Approve / Reject), comment textarea (required for reject), submit button. On submit, call `approval.approve` or `approval.reject` tRPC procedure.
4. On approval/rejection, send status update email to the rep via Resend; include the approver's comment and a link back to the quote.
5. Send approval-request email to the next sequential approver (or all parallel approvers) including a direct deep link to the review screen.
6. Update `ApprovalStatusBadge` polling interval: 15 seconds when quote is in `pending_approval` state; pauses when not in approval flow.
