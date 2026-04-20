# T027 — Internal Quote Collaboration

**Sprint:** [Sprint 06](../sprints/sprint-06.md) | **Phase:** 3

## User Stories
- As a rep, I can leave a comment on a specific line item or section to ask a colleague a question.
- As a reviewer, I receive an @mention notification with a direct link to the relevant part of the quote.
- As a manager, I can formally request a review from a specific colleague and see the structured feedback.

## Definition of Success
- Comments can be left on: the quote level, a specific line item, or a named section.
- @mention detection in comment text triggers email notification to the mentioned user.
- Internal review request sends the reviewer a view-only link and a structured feedback form; feedback is logged on the quote.
- Comment threads are displayed in a side panel on the quote builder page, sorted by thread.
- A "lock for review" toggle prevents editing of the quoted line items while a review is pending.
- All comments and resolutions are logged in the quote activity feed.

## Files & Objects Touched
- `packages/db/prisma/schema.prisma` — `QuoteComment`, `QuoteReviewRequest`
- `apps/web/components/quotes/CommentThread.tsx`
- `apps/web/components/quotes/CommentSidePanel.tsx`
- `apps/web/app/quotes/[quoteId]/review/[token]/page.tsx` — reviewer view-only page
- `packages/api/src/routers/comment.ts`
- `packages/email/src/templates/mention-notification.tsx`

## Methodology
1. Define `QuoteComment`: `quote_id`, `line_item_id` (nullable), `section_id` (nullable), `body`, `author_user_id`, `parent_comment_id` (for threading), `resolved`, `created_at`.
2. Define `QuoteReviewRequest`: `quote_id`, `reviewer_user_id`, `requester_user_id`, `token` (UUID), `status` (pending | completed | dismissed), `feedback` (JSON), `created_at`, `completed_at`.
3. Build `CommentSidePanel` component: filterable by context (all | line item | section); shows threaded comments with reply box. Inline @mention detection via `@username` pattern triggers a user search dropdown.
4. On comment save, parse `@mentions`; for each mention, queue an email notification with a direct link to the quote + comment anchor.
5. Review request flow: generates a `token`-signed view-only URL; sends email to reviewer. Reviewer page shows read-only quote + a feedback form (structured: rating, concerns, approval recommendation). On submit, saves feedback to `QuoteReviewRequest.feedback` and notifies requester.
6. "Lock for review" toggle: sets `Quote.review_locked = true`; the quote builder disables editing of line items while locked. Auto-unlocks when review is completed or dismissed.
