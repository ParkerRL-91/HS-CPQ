# T006 — PDF Generation

**Sprint:** [Sprint 02](../sprints/sprint-02.md) | **Phase:** 1

## User Stories
- As a rep, when I finalize a quote a professional PDF is generated that I can send to the prospect.
- As a prospect, the PDF contains my company's name, rep contact details, all line items with pricing, and totals — formatted cleanly.
- As the system, PDF generation does not block the API response and the resulting file is stored durably.

## Definition of Success
- PDF is generated via Puppeteer running in a serverless function or background job.
- Dynamic tokens resolve correctly: `{{company.name}}`, `{{contact.first_name}}`, `{{owner.name}}`, `{{quote.total}}`, `{{quote.valid_until}}`, line item table.
- PDF is stored in Cloudflare R2/S3; the public URL is saved to `Quote.pdf_url`.
- Generation job completes within 10 seconds for a quote with up to 20 line items.
- Reps can re-generate the PDF after any edit; the new URL overwrites the old.

## Files & Objects Touched
- `packages/pdf/src/renderer.ts` — Puppeteer launch, HTML → PDF pipeline
- `packages/pdf/src/templates/default.html` — base HTML quote template
- `packages/pdf/src/token-resolver.ts` — resolves `{{token}}` patterns against deal/company/quote data
- `packages/storage/src/r2.ts` — upload helper for Cloudflare R2
- `packages/jobs/src/generate-pdf.ts` — Inngest job definition
- `apps/web/app/api/quotes/[quoteId]/pdf/route.ts` — trigger endpoint

## Methodology
1. Author `default.html`: static HTML + CSS template with placeholder tokens. Include header (logo, company name), line item table (name, qty, unit price, discount, total), summary block (subtotal, discount, total), and footer (rep info, valid until, terms).
2. Implement `token-resolver.ts`: takes the HTML string and a data context object; uses regex to replace `{{path.to.value}}` tokens with resolved values; escapes HTML.
3. Implement `renderer.ts`: launches Puppeteer (or Playwright), navigates to a data: URL of the resolved HTML, calls `page.pdf()` with A4/Letter settings, returns the PDF buffer.
4. Implement `r2.ts` upload: streams the buffer to R2, returns a signed or public URL.
5. Define an Inngest job `generate-pdf` that takes a `quoteId`, fetches all required data, runs the renderer, uploads, and updates `Quote.pdf_url`.
6. `POST /api/quotes/[quoteId]/pdf` enqueues the Inngest job and returns `202 Accepted`; the client polls `quote.get` until `pdf_url` is populated.
