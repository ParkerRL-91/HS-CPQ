# T026 — Quote Engagement Tracking

**Sprint:** [Sprint 06](../sprints/sprint-06.md) | **Phase:** 3

## User Stories
- As a rep, I am notified when a prospect first opens my quote so I can follow up at the right moment.
- As a rep, I can see how much time the prospect spent on each section of the quote so I understand where their concerns are.
- As a manager, I can see aggregate engagement data across all sent quotes to identify patterns.

## Definition of Success
- First open event is recorded with timestamp and fires a rep notification within 60 seconds.
- Section-level time tracking records dwell time per named section (using IntersectionObserver on the hosted quote page).
- Re-open count, scroll depth, and sharing signals (multiple IP addresses) are tracked.
- All events written to `QuoteEngagementEvent` table.
- Rep-facing engagement panel on the quote detail page shows: first open, re-open count, sections viewed, time per section, last activity.

## Files & Objects Touched
- `apps/web/app/q/[token]/page.tsx` — hosted quote page (tracking instrumentation)
- `apps/web/app/q/[token]/events/route.ts` — event collection endpoint
- `packages/db/prisma/schema.prisma` — `QuoteEngagementEvent`
- `apps/web/components/quotes/EngagementPanel.tsx` — rep-facing engagement summary
- `packages/jobs/src/first-open-notification.ts` — notify rep on first open
- `packages/api/src/routers/quote.ts` — `getEngagementSummary` procedure

## Methodology
1. Define `QuoteEngagementEvent`: `quote_id`, `event_type` (first_open | reopen | section_view | scroll_depth | accept | request_changes), `session_id` (UUID generated per page load), `section_id`, `duration_seconds`, `ip_hash` (hashed for privacy), `user_agent`, `created_at`.
2. On hosted quote page load, generate a `session_id`, send `first_open` or `reopen` event via a fire-and-forget `POST /q/[token]/events`.
3. Use `IntersectionObserver` on each named quote section; when a section enters the viewport, start a timer; when it leaves, stop the timer and send a `section_view` event with `duration_seconds`.
4. Track scroll depth: record `scroll_depth` event at 25%, 50%, 75%, 100% thresholds (each threshold only once per session).
5. Event endpoint: validates token, maps to quote ID, inserts `QuoteEngagementEvent` record asynchronously. Returns `204` immediately; use a non-blocking write.
6. `first-open-notification` Inngest job: triggered on the first `first_open` event per quote; sends a rep notification email "Your quote was just opened by [company name]" with engagement summary link.
