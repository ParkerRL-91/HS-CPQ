# T043 — Documentation & In-App Onboarding

**Sprint:** [Sprint 08](../sprints/sprint-08.md) | **Phase:** 4

## User Stories
- As a new admin, I can follow an in-app setup wizard that walks me through connecting my HubSpot portal, configuring products, and sending my first quote.
- As a developer building an integration, I have access to API documentation that describes all available endpoints and their schemas.
- As a new rep, I can access contextual help from anywhere in the product without leaving the page.

## Definition of Success
- Setup wizard (5–7 steps): OAuth connection → product sync → first product pricing config → template selection → first quote creation → send quote. Each step validates completion before enabling the next.
- Admin guide covers: installation, all admin-configurable features, user management, and troubleshooting common errors.
- API docs (auto-generated from tRPC router + zod schemas) are published at `/docs/api`.
- Contextual help tooltips on complex UI elements (approval chain builder, pricing matrix, discount schedule).
- Help center link in the nav opens the admin guide in a new tab.

## Files & Objects Touched
- `apps/web/app/setup/page.tsx` — setup wizard route
- `apps/web/components/setup/SetupStep.tsx`
- `apps/web/components/setup/SetupWizard.tsx`
- `packages/db/prisma/schema.prisma` — `Portal.setup_completed_steps` (JSON array)
- `apps/web/components/ui/HelpTooltip.tsx` — reusable contextual help component
- `apps/docs/` — API documentation site (Next.js)

## Methodology
1. Define setup steps as an ordered array with: step ID, title, description, completion check function, and action component.
2. Build `SetupWizard` component: vertical step list (like a checkout stepper); active step expanded with action content; completed steps show a green checkmark; locked steps greyed out.
3. Completion state stored in `Portal.setup_completed_steps` JSON array; wizard resumes from last incomplete step on re-visit.
4. Auto-generate API docs: use `trpc-openapi` or a custom tRPC-to-OpenAPI adapter to generate an OpenAPI spec from the router definitions; publish with Swagger UI or Redoc.
5. `HelpTooltip` component: wraps any label/icon; renders a `Popover` with a description string on hover. Used inline throughout the admin UI on complex fields.
6. Write admin guide as MDX files in `apps/docs/`; deploy as a Next.js docs site at `/docs`. Structure: Getting Started, Pricing Configuration, Approvals, Subscriptions, Templates, User Management, Troubleshooting.
