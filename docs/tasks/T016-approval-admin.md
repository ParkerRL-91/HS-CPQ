# T016 — Approval Admin (Rule Builder)

**Sprint:** [Sprint 04](../sprints/sprint-04.md) | **Phase:** 2

## User Stories
- As an admin, I can define approval triggers and chains in a drag-and-drop UI without writing code.
- As an admin, I can assign approvers by specific user, role, or dynamic relationship (e.g., deal owner's manager).
- As an admin, I can scope approval rules to specific product families, customer segments, or deal types.

## Definition of Success
- Admin can create, edit, reorder, enable/disable, and delete approval chains from a dedicated admin panel.
- Trigger rule builder supports: discount threshold, deal size, product presence/family, non-standard terms, customer tier.
- Approver assignment supports: static user, role-based (any user with role), dynamic (deal owner's manager from user hierarchy).
- Chain type selector: sequential, parallel, first-response, delegated.
- Changes take effect on the next quote submission; existing pending approvals are not affected.

## Files & Objects Touched
- `apps/web/app/admin/approvals/page.tsx` — approval chain list
- `apps/web/app/admin/approvals/[chainId]/page.tsx` — chain detail + step editor
- `apps/web/components/admin/ApprovalChainBuilder.tsx`
- `apps/web/components/admin/ApproverSelector.tsx`
- `packages/api/src/routers/approval-chain.ts` — tRPC CRUD for chains
- `packages/db/prisma/schema.prisma` — `ApprovalChain.steps_config`, `ApprovalChain.trigger_rules`

## Methodology
1. Build chain list page: table of all chains with name, type, trigger count, active toggle, edit/delete actions.
2. Build chain detail page: trigger rule section (reuses condition builder component from rule engine) and steps section.
3. Steps section: drag-to-reorder step cards. Each card has an approver selector and a "delegation enabled" toggle.
4. `ApproverSelector` component: toggle between static user (search input), role-based (dropdown of available roles), and dynamic (dropdown of dynamic resolver types: "Deal owner's manager", "VP of Sales", "Finance Lead").
5. Store the resolved chain config as `steps_config` JSON on the `ApprovalChain` record; the engine reads this JSON at approval time to instantiate `ApprovalStep` records.
6. Implement delegation rule storage: each user can have an `ApprovalDelegation` record with `delegate_user_id`, `start_date`, `end_date`; admin UI to manage this in the user settings page.
