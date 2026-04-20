# T034 — Guided Selling Wizard

**Sprint:** [Sprint 07](../sprints/sprint-07.md) | **Phase:** 4

## User Stories
- As a rep, before building a quote I can launch a wizard that asks me qualifying questions and narrows the product catalog to the right options.
- As an admin, I can configure the wizard questions, branching logic, and which answers trigger which products.
- As a rep, I can skip the wizard at any time and proceed directly to the full product catalog.

## Definition of Success
- Admin can create a `GuidedSellingFlow` with a question tree: question types (single-choice, multi-choice, text, numeric, date), branching conditions, and product filter mappings.
- Wizard renders as a step-by-step modal in the quote builder; each answer progresses to the next question or branches.
- At completion, the product catalog is filtered to matching products; high-confidence matches are auto-added as suggested line items.
- Rep can "Skip Wizard" at any step; wizard state is preserved in a `GuidedSellingSession` record for analytics.
- Wizard completion records are queryable for analytics (which branches reps take most often).

## Files & Objects Touched
- `packages/db/prisma/schema.prisma` — `GuidedSellingFlow`, `GuidedSellingSession`
- `apps/web/app/admin/guided-selling/[flowId]/edit/page.tsx` — flow editor
- `apps/web/components/admin/QuestionTreeBuilder.tsx`
- `apps/web/components/quote-builder/GuidedSellingWizard.tsx` — wizard modal
- `packages/api/src/routers/guided-selling.ts`

## Methodology
1. Define `GuidedSellingFlow` schema: `portal_id`, `name`, `active`, `questions` (JSON tree: `{id, text, type, options[], branches: [{answer_match, next_question_id}], product_filters: [{field, op, value}]}`).
2. Build admin `QuestionTreeBuilder`: drag-to-reorder questions, branching config (answer → next question), product filter conditions per answer. Preview renders the wizard in read-only mode.
3. Build `GuidedSellingWizard` modal: step-through UI; each question renders the appropriate input type; "Next" evaluates branches; stores answers in local state.
4. At wizard completion, call `guided-selling.resolveProducts` tRPC procedure: server applies all product filters from the accumulated answers; returns matching products sorted by confidence score. High-confidence products (all filters matched) are marked for auto-add.
5. Auto-add suggestions: show a review step before closing the wizard ("We'll add these products — confirm or adjust"); on confirmation, calls `quote.addLineItem` for each suggested product.
6. Record `GuidedSellingSession` with `answers` JSON and `completed_at`; link to the quote.
