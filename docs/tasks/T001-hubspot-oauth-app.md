# T001 — HubSpot OAuth App Registration & Token Flow

**Sprint:** [Sprint 01](../sprints/sprint-01.md) | **Phase:** 1

## User Stories
- As a portal admin, I can install the CPQ app from HubSpot so that my portal's CRM data is accessible to the platform.
- As the system, I can refresh expired OAuth tokens automatically so that API calls never fail due to stale credentials.
- As a developer, I can scope the OAuth install to a specific portal so that all subsequent API calls are correctly tenant-isolated.

## Definition of Success
- A HubSpot Public App is registered with the required OAuth scopes (crm.objects.deals, crm.objects.contacts, crm.objects.companies, crm.objects.products, crm.objects.line_items, crm.objects.quotes).
- The install flow completes end-to-end: user is redirected to HubSpot, grants scopes, tokens are exchanged and persisted to the `Portal` table.
- Token refresh runs automatically before expiry; refresh failures trigger an alert and a re-auth prompt.
- All API client calls attach the correct portal token based on the active session's `portal_id`.

## Files & Objects Touched
- `apps/web/app/api/auth/hubspot/` — install, callback, and refresh route handlers
- `packages/db/prisma/schema.prisma` — `Portal` model (`hubspot_portal_id`, `oauth_access_token`, `oauth_refresh_token`, `token_expires_at`)
- `packages/hubspot/client.ts` — HubSpot Node.js SDK wrapper with per-portal token injection
- `packages/hubspot/token-manager.ts` — token refresh logic and storage
- `.env` — `HUBSPOT_CLIENT_ID`, `HUBSPOT_CLIENT_SECRET`, `HUBSPOT_REDIRECT_URI`

## Methodology
1. Register a HubSpot Public App in the developer portal; note client ID/secret.
2. Implement `/api/auth/hubspot/install` — builds the OAuth authorization URL with required scopes and redirects.
3. Implement `/api/auth/hubspot/callback` — exchanges the code for access/refresh tokens, upserts the `Portal` record.
4. Build `token-manager.ts`: checks `token_expires_at` before each API call; if within 5 minutes of expiry, refreshes proactively using the refresh token endpoint.
5. Wrap the HubSpot SDK client so it accepts a `portal_id` and injects the correct token per request.
6. Write integration tests against a HubSpot sandbox portal that exercise the full install → callback → refresh cycle.
