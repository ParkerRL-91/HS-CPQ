# T002 — Database Schema & Multi-Tenant Foundation

**Sprint:** [Sprint 01](../sprints/sprint-01.md) | **Phase:** 1

## User Stories
- As the system, all database queries are automatically scoped to the correct portal so cross-tenant data leakage is impossible.
- As a developer, I can run migrations deterministically so the schema stays in sync across all environments.
- As the platform, I can scale to multiple portals without any schema changes.

## Definition of Success
- Prisma schema defines all core entities from the data model (Portal, Product, PricingMatrix, DiscountSchedule, PriceRule, SalesAgreement, AgreementLineItem, Quote, QuoteOptionGroup, LineItem, ApprovalRequest, ApprovalStep, ApprovalChain, Subscription, SubscriptionAmendment, QuoteTemplate, User).
- Every table includes `portal_id`, `created_at`, `updated_at`.
- Prisma middleware intercepts all queries and appends `WHERE portal_id = ?` for all non-Portal queries.
- Integration test explicitly attempts a cross-tenant read and confirms it returns zero results.
- Migrations run cleanly in CI from a blank database.

## Files & Objects Touched
- `packages/db/prisma/schema.prisma` — full entity definitions
- `packages/db/prisma/migrations/` — initial migration files
- `packages/db/src/middleware/tenant-scope.ts` — Prisma middleware enforcing `portal_id`
- `packages/db/src/client.ts` — Prisma client instantiation with middleware applied
- `packages/db/src/seed.ts` — development seed data

## Methodology
1. Author `schema.prisma` with all entities, referential integrity, and indexes on `portal_id` + foreign keys.
2. Run `prisma migrate dev` to generate the initial migration; commit migration files.
3. Implement `tenant-scope.ts` as a Prisma middleware: reads `portal_id` from a per-request context (AsyncLocalStorage) and appends it as a `where` condition on all `findMany`, `findFirst`, `findUnique`, `update`, `delete` operations.
4. Wrap the Prisma client in `client.ts` with the middleware applied at initialization; export a singleton.
5. Write an integration test that creates records for two portals, then queries with portal A's context and confirms portal B's records are not returned.
6. Add seed data for local development with a test portal, sample products, and a sample user.
