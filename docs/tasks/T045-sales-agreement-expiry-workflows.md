# T045 — Sales Agreement Expiry Workflows

**Sprint:** [Sprint 09](../sprints/sprint-09.md) | **Phase:** 5

## User Stories
- As an account owner, I receive a notification 60 and 30 days before a Sales Agreement expires so I can start the renewal conversation.
- As the system, a renewal agreement is automatically generated at the 30-day mark with the current terms as a baseline.
- As an admin, I can review and approve the renewal agreement before it becomes active for the account.

## Definition of Success
- Daily Inngest job scans for agreements expiring within 60 and 30 days; sends notifications (once per window per agreement).
- At 30 days, a new `SalesAgreement` in `draft` status is created as a copy of the expiring agreement with updated `start_date`/`end_date`.
- Account owner receives an email with a link to review and submit the renewal agreement for approval.
- Renewal agreement goes through the standard agreement approval flow before activating.
- `AgreementNotificationLog` table prevents duplicate notifications.

## Files & Objects Touched
- `packages/jobs/src/agreement-expiry-scan.ts` — daily Inngest job
- `packages/subscriptions/src/agreement-renewal-creator.ts` — creates renewal agreement draft
- `packages/db/prisma/schema.prisma` — `AgreementNotificationLog`
- `packages/email/src/templates/agreement-expiry-notice.tsx`
- `packages/email/src/templates/renewal-agreement-ready.tsx`
- `packages/api/src/routers/sales-agreement.ts` — `submitForRenewalApproval` procedure

## Methodology
1. Define `AgreementNotificationLog`: `agreement_id`, `window_days` (60 | 30), `sent_at`, `recipient_user_id`. Unique constraint on `(agreement_id, window_days)`.
2. `agreement-expiry-scan` daily job: queries active agreements with `end_date` within 60 ± 1 day or 30 ± 1 day; excludes agreements already logged; sends notifications and inserts log records.
3. 60-day notification: email to account owner with agreement summary, expiry date, and a prompt to start renewal discussions. No automatic draft created yet.
4. 30-day notification + `agreement-renewal-creator.ts`: creates a new `SalesAgreement` (status = draft, `parent_agreement_id` = expiring agreement ID, `start_date` = expiring agreement's `end_date + 1 day`, `end_date` = new term, same line items). Sends a "Renewal agreement ready for review" email with link to the draft.
5. Account owner reviews the draft, adjusts terms if needed, and submits via `submitForRenewalApproval` procedure — this transitions the draft to `pending_approval` and triggers the agreement approval chain.
6. On approval activation: the new agreement's `status` is set to `active`; the original agreement's `status` is set to `expired`. The account's contracted prices automatically update to the new agreement's terms.
