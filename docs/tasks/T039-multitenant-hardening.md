# T039 — Multi-Tenant Hardening

**Sprint:** [Sprint 08](../sprints/sprint-08.md) | **Phase:** 4

## User Stories
- As the platform, no query can return data belonging to a different portal under any code path.
- As an operator, I can be confident that load testing confirms tenant isolation holds under concurrent traffic.
- As a security reviewer, I can see an audit of all database queries confirming portal_id is enforced.

## Definition of Success
- Integration test suite explicitly attempts cross-tenant data access across all resource types (quotes, products, subscriptions, approvals) and confirms zero results are returned.
- Prisma middleware tenant-scope enforcement is verified via a code review checklist that maps every Prisma query in the codebase.
- Load test with 10 concurrent portals at 50 requests/second shows no data leakage in response payloads.
- Any Prisma query that bypasses middleware (e.g., `$queryRaw`) is identified, audited, and wrapped with an explicit portal_id filter.
- Security findings from the audit are documented and resolved before marketplace listing.

## Files & Objects Touched
- `packages/db/src/middleware/tenant-scope.ts` — middleware review and hardening
- `tests/integration/tenant-isolation.test.ts` — cross-tenant access test suite
- `tests/load/tenant-concurrency.k6.ts` — k6 load test script
- `packages/db/src/client.ts` — raw query audit
- `docs/security/tenant-isolation-audit.md` — audit findings document

## Methodology
1. Enumerate all Prisma model operations in the codebase (`prisma.X.findMany`, `findFirst`, `findUnique`, `update`, `delete`); confirm each is routed through the middleware. Flag any that call `$queryRaw` or `$executeRaw`.
2. For each raw SQL query identified: add an explicit `WHERE portal_id = $1` clause and parameterize it with the request context's `portal_id`; add a comment explaining why a raw query is needed.
3. Extend the middleware to also enforce `portal_id` on nested includes (related records); verify that a query for a quote also scopes its line items to the same portal.
4. Write `tenant-isolation.test.ts`: create portal A with quotes, products, approvals; query all resource types from portal B's context; assert empty results for all.
5. Write k6 load test: 10 virtual portals, 50 RPS each; validate response payloads contain only the requesting portal's data using JSON assertions.
6. Document all findings in `tenant-isolation-audit.md`; track remediation status per finding; mark complete before marketplace prep (T041).
