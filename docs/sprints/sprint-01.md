# Sprint 01 — Foundation & Infrastructure

**Phase:** 1 | **Weeks:** 1–4

## Goal
Establish the technical foundation: HubSpot OAuth app wired to a multi-tenant PostgreSQL schema, product catalog synced from HubSpot, and a working basic pricing engine. By the end of this sprint, the system can authenticate with a HubSpot portal, read products, and return priced line items for simple pricing methods.

## Summary
This sprint creates everything that nothing else can be built on top of without. OAuth gives us the CRM access token. The database schema enforces multi-tenancy via `portal_id` from day one — no retrofit later. Product sync bridges the HubSpot Products Library into CPQ-extended records. The pricing engine is implemented as pure TypeScript functions so it is fully testable before any UI touches it.

## Tasks
| ID | Task | Owner |
|----|------|-------|
| [T001](../tasks/T001-hubspot-oauth-app.md) | HubSpot OAuth App Registration & Token Flow | |
| [T002](../tasks/T002-database-schema.md) | Database Schema & Multi-Tenant Foundation | |
| [T003](../tasks/T003-product-catalog-sync.md) | Product Catalog Sync from HubSpot | |
| [T004](../tasks/T004-basic-pricing-engine.md) | Basic Pricing Engine | |

## Definition of Done
- A test portal can complete the OAuth flow and tokens are stored and refreshed automatically.
- Prisma migrations run cleanly; all tables include `portal_id`, `created_at`, `updated_at`.
- HubSpot products sync to the `Product` table with CPQ extension fields.
- Pricing engine unit tests pass for: list price, per unit, flat fee, block pricing, percent-of-total.
