# T048 — Partner Portal MVP

**Sprint:** [Sprint 10](../sprints/sprint-10.md) | **Phase:** 5

## User Stories
- As a channel partner, I can log in to a self-service portal and create quotes for my assigned products within my approved discount tiers.
- As an admin, I can configure which products and discount limits each partner can access.
- As the system, partner quotes flow through the standard approval engine with partner-specific approval chains.

## Definition of Success
- `PartnerOrg` and `PartnerUser` models isolate partner authentication from internal users.
- Partners can only see and select products in their assigned `PartnerProductAccess` set.
- Discount caps from the partner's `PartnerProductAccess` record are enforced by the pricing engine; exceeding them triggers an approval.
- Partner-submitted quotes are visible to the portal admin in a dedicated "Channel Quotes" view.
- Partner portal is available at a configurable subdomain (e.g., `partners.yourdomain.com`) with optional branding per partner org.

## Files & Objects Touched
- `packages/db/prisma/schema.prisma` — `PartnerOrg`, `PartnerUser`, `PartnerProductAccess`
- `apps/partner-portal/` — standalone Next.js app for the partner portal
- `packages/auth/src/partner-auth.ts` — partner authentication (separate from internal auth)
- `packages/pricing/src/discount-waterfall.ts` — enforce partner discount caps
- `apps/web/app/admin/partners/page.tsx` — partner management admin UI
- `packages/api/src/routers/partner.ts` — partner-scoped tRPC router

## Methodology
1. Define `PartnerOrg`: `portal_id`, `name`, `slug` (for subdomain), `branding` (JSON), `status`. Define `PartnerUser`: `partner_org_id`, `email`, `name`, `password_hash` (separate credential store from internal users). Define `PartnerProductAccess`: `partner_org_id`, `product_id`, `max_discount_pct`.
2. Build `apps/partner-portal` as a separate Next.js app sharing the `packages/` monorepo libraries but with its own layout, auth, and routes. Route at `/` shows the partner's quote list; `/quotes/new` opens the CPQ configurator scoped to allowed products.
3. Implement `partner-auth.ts`: email/password authentication with JWT session; passwords hashed with bcrypt. Admin invite flow: admin creates a partner user account; partner receives a "set password" email.
4. Partner product catalog: the product search in the partner quote builder queries only products in `PartnerProductAccess` for the authenticated partner org.
5. Pricing engine enforcement: pass `partnerDiscountCap` from `PartnerProductAccess` into the discount waterfall; if manual discount + waterfall exceeds the cap, trigger an approval with the partner-specific chain.
6. Admin partner management UI: create/edit partner orgs, manage users, assign product access with discount caps, view all channel quotes with status and rep assignment.
