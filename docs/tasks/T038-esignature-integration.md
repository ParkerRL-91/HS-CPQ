# T038 — E-Signature Integration

**Sprint:** [Sprint 08](../sprints/sprint-08.md) | **Phase:** 4

## User Stories
- As a rep, I can send a quote for e-signature via DocuSign or Adobe Sign directly from the CPQ platform.
- As a prospect, I receive a signature request email and can sign the document without a DocuSign/Adobe Sign account.
- As the system, when the document is signed the HubSpot Deal stage is automatically updated.

## Definition of Success
- Admin can configure which e-signature provider is active for the portal (DocuSign, Adobe Sign, or click-to-accept only).
- From the quote detail page, rep can initiate a signature request: maps signers from deal contacts, selects signature fields (pre-placed on the PDF or via the provider's tagging interface).
- Signer receives an email from the configured provider with the quote PDF.
- Provider webhooks deliver status updates (viewed, signed, declined); status is stored on the `Quote` and displayed in the sidebar.
- On completion, the signed PDF is downloaded and stored in R2; `Quote.signed_pdf_url` is set; HubSpot Deal is updated.

## Files & Objects Touched
- `packages/esignature/src/docusign.ts` — DocuSign envelope creation and webhook handling
- `packages/esignature/src/adobe-sign.ts` — Adobe Sign agreement creation and webhook handling
- `packages/esignature/src/provider.ts` — provider abstraction layer
- `apps/web/app/api/webhooks/esignature/route.ts` — webhook receiver
- `packages/db/prisma/schema.prisma` — `Quote.esignature_envelope_id`, `Quote.esignature_status`, `Quote.signed_pdf_url`
- `apps/web/components/quotes/ESignaturePanel.tsx`

## Methodology
1. Define `provider.ts` abstraction: `createEnvelope(quoteId, signers[], pdfBuffer)` and `getStatus(envelopeId)` — both return a normalized `EnvelopeResult` type. DocuSign and Adobe Sign implementations conform to this interface.
2. `docusign.ts`: use DocuSign eSign REST API; create an envelope with the quote PDF as document, map signers (name, email), and pre-place signature tabs using anchor strings embedded in the PDF template.
3. `adobe-sign.ts`: use Adobe Sign REST API; create a transient document from the PDF, then create an agreement with the signers list.
4. Webhook endpoint validates the provider's HMAC signature; on `envelope.completed` / `agreement.SIGNED` event, downloads the signed PDF, uploads to R2, updates `Quote.signed_pdf_url` and `Quote.esignature_status`, triggers HubSpot Deal stage update.
5. Build `ESignaturePanel` component: shows current signature status with provider icon, signer list and each signer's status (pending / viewed / signed / declined), and a "Resend" button for pending signers.
6. Handle `declined` status: notify rep immediately; set quote back to `approved` so rep can revise and re-send.
