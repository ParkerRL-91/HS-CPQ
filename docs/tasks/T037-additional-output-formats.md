# T037 — Word & Excel Quote Export

**Sprint:** [Sprint 08](../sprints/sprint-08.md) | **Phase:** 4

## User Stories
- As a rep, I can export a quote as a Word document (.docx) for prospects who need to edit or redline it.
- As a rep, I can export a quote as an Excel spreadsheet (.xlsx) for prospects who need the data in a structured format for their procurement process.
- As the system, both export formats accurately reflect the current template and line items.

## Definition of Success
- `.docx` export uses the `docx` npm library; applies the same dynamic tokens as the PDF template; preserves section headings and line item tables.
- `.xlsx` export uses `exceljs`; produces a structured spreadsheet with: header info (company, rep, valid until), line item table (product, qty, list price, discount, unit price, extended price), and a totals section.
- Both export jobs run server-side; files are returned as downloads or stored to R2 and linked.
- Export is accessible from the quote detail page under an "Export" dropdown.

## Files & Objects Touched
- `packages/export/src/docx-exporter.ts` — Word document generation
- `packages/export/src/xlsx-exporter.ts` — Excel spreadsheet generation
- `packages/storage/src/r2.ts` — file upload
- `apps/web/app/quotes/[quoteId]/export/route.ts` — export trigger endpoint
- `packages/jobs/src/generate-export.ts` — Inngest job

## Methodology
1. `docx-exporter.ts`: use the `docx` npm package to programmatically build a Word document. Walk the active template's TipTap JSON; map text blocks to `Paragraph` nodes, line item tables to `Table` nodes with `TableRow`/`TableCell` construction. Resolve tokens using the same `token-resolver.ts` as the PDF pipeline.
2. `xlsx-exporter.ts`: use `exceljs` to build a workbook with two sheets — "Quote" (formatted with merged cells, bold headers, and column auto-sizing) and "Line Items" (flat data table, all columns, for easy manipulation). Apply number formats to price and discount columns.
3. Both exporters return a `Buffer`; the Inngest job uploads the buffer to R2 and saves the download URL to `Quote.docx_url` and `Quote.xlsx_url` respectively.
4. Build an "Export" dropdown button on the quote detail page with options "PDF", "Word (.docx)", "Excel (.xlsx)". Clicking triggers the appropriate export job and shows a loading state until the URL is available.
5. For `.docx`, also support a "Clean copy" option (no tracked changes, no conditional-content markers) vs. a "Redline copy" option (tracked changes mode enabled) for negotiation workflows.
