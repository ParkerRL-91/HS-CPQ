# T036 — Multi-Option Quotes

**Sprint:** [Sprint 08](../sprints/sprint-08.md) | **Phase:** 4

## User Stories
- As a rep, I can create a quote with multiple pricing scenarios (Good/Better/Best) so the prospect can choose the option that fits their budget.
- As a prospect, I can compare options side by side on the hosted quote page and select my preferred one.
- As the system, once a prospect selects an option the non-selected options are archived and the selected line items become the active quote.

## Definition of Success
- Quote builder supports 2–4 named option groups; each group has its own independent line items.
- One group is designated as the "primary" (recommended) option, visually highlighted.
- Hosted quote page renders a comparison table at the top showing all options' totals, then each option's full line item detail below.
- On prospect selection, `QuoteOptionGroup.status` is updated (selected | archived); selected line items are promoted to the active quote state.
- Each option group can have its own approval state if discount levels differ.

## Files & Objects Touched
- `packages/db/prisma/schema.prisma` — `QuoteOptionGroup`
- `apps/web/components/quote-builder/OptionGroupManager.tsx`
- `apps/web/app/q/[token]/page.tsx` — multi-option rendering and selection flow
- `packages/api/src/routers/quote.ts` — `addOptionGroup`, `removeOptionGroup`, `selectOption` procedures
- `apps/web/app/q/[token]/select/route.ts` — prospect option selection handler

## Methodology
1. Extend `QuoteOptionGroup` schema: `quote_id`, `name`, `is_primary`, `status` (active | selected | archived), `sort_order`.
2. `LineItem` gains `option_group_id` foreign key; all line items belong to an option group (default group created for standard quotes).
3. Build `OptionGroupManager` component in the quote builder: tab bar at the top for each group; each tab shows the group's own `LineItemRow` list and subtotal; reps can add/remove line items independently per group.
4. Quote builder header bar: "Add Option" button creates a new group; groups are renameable; "Set as Primary" toggle.
5. Hosted quote page: if `quote.option_groups.length > 1`, render a comparison table header (group names + totals + "Select" buttons). Below, each group's full line item table in a tabbed or stacked layout.
6. `POST /q/[token]/select`: receives `option_group_id`, sets that group's `status = selected`, sets all other groups to `archived`, records a `QuoteEngagementEvent` with `event_type = option_selected`, triggers HubSpot sync.
