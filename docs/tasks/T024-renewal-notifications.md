# T024 — Renewal Notifications

**Sprint:** [Sprint 05](../sprints/sprint-05.md) | **Phase:** 3

## User Stories
- As a rep, I receive email notifications at 90, 60, and 30 days before a subscription renewal so that I can act in time.
- As a manager, I receive a summary digest of upcoming renewals in my team so that I can prioritize coaching.
- As the system, notifications are sent exactly once per window and are not re-sent on system restarts.

## Definition of Success
- Renewal notifications fire at exactly 90, 60, and 30 days before `Subscription.end_date`.
- Notifications are deduplicated: a `RenewalNotificationLog` table records sent notifications; the job skips already-sent entries.
- Each notification includes: subscription summary, renewal quote link (if already generated), days remaining, and a prompt to action.
- Manager digest runs weekly on Mondays summarizing all their direct reports' upcoming renewals (30/60/90-day view).
- Notification delivery confirmed via Resend webhook; failed sends are retried.

## Files & Objects Touched
- `packages/jobs/src/renewal-notifications.ts` — daily notification job
- `packages/jobs/src/manager-renewal-digest.ts` — weekly digest job
- `packages/email/src/templates/renewal-reminder.tsx`
- `packages/email/src/templates/manager-renewal-digest.tsx`
- `packages/db/prisma/schema.prisma` — `RenewalNotificationLog`

## Methodology
1. Define `RenewalNotificationLog`: `subscription_id`, `window_days` (90 | 60 | 30), `sent_at`, `recipient_user_id`. Unique constraint on `(subscription_id, window_days)` prevents duplicates.
2. `renewal-notifications` daily job: for each window (90, 60, 30 days), query subscriptions where `end_date` is within that window ± 1 day; exclude subscriptions already logged for that window. Send email; insert log record.
3. Renewal reminder email template: subscription details (product, quantity, ARR, contract dates), renewal quote CTA (if `renewal_quote_id` exists), urgency indicator (color-coded by days remaining), rep action checklist.
4. `manager-renewal-digest` weekly job (Monday 8am): for each manager, fetch all active subscriptions owned by their direct reports expiring in the next 90 days; group by window; render digest email sorted by ARR value descending.
5. Register Resend delivery webhook: on `email.bounced` or `email.failed`, log the failure and enqueue a retry via Inngest with 2-hour delay (max 2 retries).
