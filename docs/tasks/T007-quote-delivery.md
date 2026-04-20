# T007 — Quote Delivery (Hosted Link + Email)

**Sprint:** [Sprint 02](../sprints/sprint-02.md) | **Phase:** 1

## User Stories
- As a rep, I can send the quote to a prospect via email with one click so that the delivery is fast and tracked.
- As a prospect, I receive a clean email with a direct link to view the quote, as well as the PDF attached.
- As a prospect, I can view the quote on a hosted web page and accept it with a single click.
- As a rep, I am notified immediately when the prospect accepts the quote.

## Definition of Success
- Hosted quote page (`/q/[token]`) is publicly accessible, renders the quote, and has a prominent "Accept Quote" button.
- Email is sent via Resend with the company logo, rep greeting, a link to the hosted page, and the PDF attached.
- Click-to-accept records a `QuoteEngagementEvent` with `event_type = accepted`, updates `Quote.status = accepted`, triggers a HubSpot Deal stage update.
- Rep receives an email notification (via Resend) within 30 seconds of acceptance.
- Hosted quote link is token-signed (JWT or UUID) and does not expose internal IDs.

## Files & Objects Touched
- `apps/web/app/q/[token]/page.tsx` — public hosted quote page
- `apps/web/app/q/[token]/accept/route.ts` — acceptance POST handler
- `packages/email/src/templates/quote-delivery.tsx` — React Email template
- `packages/email/src/templates/acceptance-notification.tsx`
- `packages/email/src/sender.ts` — Resend client wrapper
- `packages/db/prisma/schema.prisma` — `Quote.delivery_token`, `Quote.status`, `QuoteEngagementEvent`

## Methodology
1. On quote finalization, generate a `delivery_token` (UUID v4) and store on the `Quote` record.
2. Build the hosted quote page (`/q/[token]`): server-side fetches quote by token, renders line items, totals, and terms. Include "Accept Quote" and "Request Changes" buttons.
3. Implement `POST /q/[token]/accept`: validates token, sets `Quote.status = accepted`, records engagement event, fires HubSpot Deal stage update via CRM API, sends acceptance notification email to rep.
4. Build React Email templates using `@react-email/components`: delivery email includes hero section, quote summary table (top 5 line items + total), CTA button linking to hosted page, and PDF attachment.
5. Wire Resend client in `sender.ts`; handle Resend API errors with retry logic (3 attempts).
6. "Request Changes" button opens a modal form; on submit, POSTs comments to `Quote.hubspot_deal_id` as a HubSpot note.
