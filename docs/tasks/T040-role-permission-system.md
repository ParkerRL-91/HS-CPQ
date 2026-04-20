# T040 — Role & Permission System

**Sprint:** [Sprint 08](../sprints/sprint-08.md) | **Phase:** 4

## User Stories
- As an admin, I can assign users to roles (Rep, Manager, Admin, Finance) so that they have appropriate access to platform features and data.
- As a rep, I can only see and edit my own quotes; I cannot access other reps' quotes or admin configuration.
- As a manager, I can see all quotes within my team but not modify pricing rules or approval chains.

## Definition of Success
- Four roles defined: Rep (own quotes, no admin), Manager (team quotes + dashboards, no rule editing), Admin (full access except billing), Finance (pricing rules + approval rules + dashboards, no quote editing).
- All tRPC procedures are guarded with role-based middleware that checks the session user's role before executing.
- Frontend hides UI elements that the user's role cannot access (menus, buttons, routes).
- Role assignment UI in admin panel; admin can assign and revoke roles per user.
- Role check tests: confirm Rep cannot call admin procedures; Manager cannot edit price rules.

## Files & Objects Touched
- `packages/auth/src/roles.ts` — role definitions and permission matrix
- `packages/api/src/middleware/require-role.ts` — tRPC middleware
- `apps/web/lib/permissions.ts` — client-side permission helper
- `apps/web/app/admin/users/page.tsx` — user role management
- `tests/unit/role-permissions.test.ts`

## Methodology
1. Define `Role` enum: `rep`, `manager`, `admin`, `finance`. Store on `User.role`.
2. Define permission matrix in `roles.ts`: a lookup of `Role → Set<Permission>` where permissions are granular strings (e.g., `quote:read_own`, `quote:read_all`, `pricing_rule:write`, `approval_chain:write`, `dashboard:view`).
3. Implement `require-role.ts` tRPC middleware: reads `ctx.session.user.role`, checks if the required permission is in the role's set; throws `TRPCError({ code: 'FORBIDDEN' })` if not. Applied as a middleware chain on each procedure.
4. `permissions.ts` client helper: `hasPermission(user, permission: string): boolean` — used in React components to conditionally render UI elements.
5. Admin user management page: table of all users with their current role, a role selector dropdown, and a save button. Admin-only route; guarded by `requireRole('admin')`.
6. Write role permission tests: for each role, attempt calls to procedures requiring a higher permission and assert `FORBIDDEN` is returned.
