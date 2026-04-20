# T042 — White-Label Branding Foundations

**Sprint:** [Sprint 08](../sprints/sprint-08.md) | **Phase:** 4

## User Stories
- As an admin, I can upload my company logo, set a primary color, and choose a font so that generated quotes match our brand.
- As a prospect, the quote PDF and hosted quote page display our company's branding, not the CPQ platform's.
- As a future commercial customer, I can brand the CPQ sidebar and quote output to look like our own product.

## Definition of Success
- Portal settings include: logo upload (stored in R2), primary color (hex, validated), secondary color, font family (from a curated list of web-safe + Google Fonts options).
- Logo and colors are applied to: PDF template header, hosted quote page header/footer, email templates.
- CSS custom properties are injected at render time from the portal's branding config; no hardcoded colors in templates.
- Branding preview renders a sample quote section with the configured branding applied.
- Branding changes take effect on the next generated quote; previously generated PDFs are not retroactively changed.

## Files & Objects Touched
- `packages/db/prisma/schema.prisma` — `Portal.branding` (JSON: logo_url, primary_color, secondary_color, font_family)
- `apps/web/app/admin/settings/branding/page.tsx` — branding settings page
- `packages/templates/src/renderer.ts` — inject branding CSS variables at render time
- `packages/pdf/src/templates/default.html` — CSS var references instead of hardcoded values
- `apps/web/app/q/[token]/page.tsx` — inject branding for hosted quote page

## Methodology
1. Extend `Portal.settings` JSON (or add a dedicated `Portal.branding` JSON column): `{ logo_url, primary_color, secondary_color, font_family, font_url }`.
2. Build branding settings page: logo upload (drag-and-drop → upload to R2 → store URL); color pickers for primary/secondary; font selector dropdown (predefined list); save button; live preview panel showing a mock quote header.
3. Extend `renderer.ts`: before rendering the template HTML, inject a `<style>` block with `:root { --brand-primary: #hex; --brand-secondary: #hex; --brand-font: 'Font Name'; }`. All template CSS uses these variables.
4. Update `default.html` template: replace all hardcoded colors and font references with CSS var() calls. Add `<link>` tag for the Google Font URL if a web font is configured.
5. Inject branding into the hosted quote page: server component reads `portal.branding`, passes it as props; page renders a `<style>` tag with the CSS vars scoped to the page root.
6. Apply branding to the Resend email templates: inline the primary color as background/button color in the React Email components (email clients require inline styles; CSS vars are not supported in email).
