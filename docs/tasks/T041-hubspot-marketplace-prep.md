# T041 — HubSpot App Marketplace Prep

**Sprint:** [Sprint 08](../sprints/sprint-08.md) | **Phase:** 4

## User Stories
- As the company, the app is listed on the HubSpot App Marketplace so that new customers can discover and install it.
- As a portal admin, installing from the marketplace starts the OAuth flow automatically.
- As the platform, HubSpot API rate limits are handled gracefully so that no portal's usage degrades another's.

## Definition of Success
- App passes HubSpot's pre-submission security checklist (OAuth scopes principle of least privilege, no credential exposure, webhook signature validation, HTTPS only).
- App listing is created with descriptions, screenshots, and feature highlights.
- Rate limit handling: all HubSpot API calls go through a per-portal request queue with token bucket rate limiting (150 req/10s per portal); rate-limited calls are automatically retried.
- Webhook reliability: all incoming webhooks are idempotently processed using event deduplication via a `WebhookEventLog` table.
- App store install URL is tested end-to-end with a new sandbox portal.

## Files & Objects Touched
- `packages/hubspot/src/rate-limiter.ts` — per-portal token bucket
- `packages/hubspot/src/webhook-validator.ts` — HMAC signature validation (hardening)
- `packages/db/prisma/schema.prisma` — `WebhookEventLog`
- `apps/web/app/api/webhooks/hubspot/route.ts` — idempotent webhook processing
- `docs/marketplace/app-listing.md` — listing content draft

## Methodology
1. Audit all OAuth scopes: remove any scope not strictly required; document justification for each remaining scope in the listing.
2. Implement `rate-limiter.ts` using a token bucket algorithm: each portal has a bucket of 150 tokens refilled every 10 seconds. Before each HubSpot API call, consume a token; if bucket is empty, queue the request and resolve when a token is available (max wait 30 seconds before failing with a descriptive error).
3. Implement `WebhookEventLog`: `hubspot_event_id` (unique), `processed_at`. On webhook receipt, check if `hubspot_event_id` exists; if so, return `200` immediately (idempotent). Otherwise, insert and process.
4. Harden webhook signature validation: verify the `X-HubSpot-Signature-v3` header using HMAC-SHA256 on every incoming webhook; reject with `401` if invalid.
5. Draft marketplace listing content: app name, tagline, category, full description (500+ words), 5 feature highlight bullets, 3–5 screenshots from the UI.
6. Complete HubSpot's pre-submission checklist; document each item's status in `docs/marketplace/`.
