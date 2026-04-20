# T032 — WYSIWYG Template Engine

**Sprint:** [Sprint 07](../sprints/sprint-07.md) | **Phase:** 4

## User Stories
- As an admin, I can build a quote template in a rich editor with drag-and-drop sections without writing HTML.
- As an admin, I can insert dynamic tokens from a picker so I don't need to remember token syntax.
- As an admin, I can preview the template with real deal data before publishing it.

## Definition of Success
- Template editor supports: text blocks, line item tables, image blocks, dividers, and section containers.
- Token picker lists all available tokens organized by category (Deal, Company, Contact, Rep, Line Items, Subscription).
- Preview mode renders the template with a selectable deal's real data.
- Template versioning: each save creates a new version; quotes always render against the template version active at generation time.
- Multiple templates per portal; templates are assignable to deal types or triggered automatically by quote attributes.

## Files & Objects Touched
- `apps/web/app/admin/templates/page.tsx` — template list
- `apps/web/app/admin/templates/[id]/edit/page.tsx` — editor page
- `apps/web/components/admin/TemplateEditor.tsx` — WYSIWYG editor (TipTap or Plate)
- `apps/web/components/admin/TokenPicker.tsx`
- `packages/templates/src/renderer.ts` — server-side template rendering (token substitution)
- `packages/db/prisma/schema.prisma` — `QuoteTemplate` (content JSON, version)
- `packages/api/src/routers/template.ts`

## Methodology
1. Choose TipTap as the editor framework (extensible, headless, good JSON serialization). Install extensions: StarterKit, Image, Table, custom TokenNode.
2. Implement `TokenNode` as a TipTap extension: renders as a non-editable chip with the token label; serializes to `{{token.path}}` in the JSON content.
3. Build `TokenPicker` component: categorized dropdown with search; on selection, inserts a `TokenNode` at the cursor position.
4. Store template content as TipTap JSON in `QuoteTemplate.content`. On version save, increment `QuoteTemplate.version` and insert a new record with the same `portal_id` and template name; old versions are preserved.
5. Implement `renderer.ts`: accepts the template JSON and a data context; walks the TipTap JSON AST, substitutes `{{token}}` nodes with resolved values, and renders to HTML for PDF generation or hosted quote display.
6. Preview mode: admin selects a deal from a search dropdown; server fetches the deal's full context and renders the template to HTML; displays in an iframe-sandboxed preview pane.
