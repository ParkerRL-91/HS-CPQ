# Sprint 08 — Commercial Readiness & Marketplace

**Phase:** 4 | **Weeks:** 29–32

## Goal
Complete Phase 4 commercial readiness: multi-option (Good/Better/Best) quotes, Word/Excel export, e-signature integration (DocuSign + Adobe Sign), multi-tenant hardening, RBAC, HubSpot App Marketplace prep, white-label branding foundations, and in-app documentation/onboarding wizard.

## Summary
This sprint prepares the product for external distribution. Multi-option quotes let prospects compare scenarios before accepting. Additional export formats satisfy enterprise procurement workflows. E-signature closes the loop on deal acceptance. The tenant isolation audit and RBAC system ensure the product is safe for multi-portal commercial use. Marketplace prep (security review checklist, rate-limit handling) and white-label configuration make the product listable and resellable.

## Tasks
| ID | Task | Owner |
|----|------|-------|
| [T036](../tasks/T036-multi-option-quotes.md) | Multi-Option Quotes | |
| [T037](../tasks/T037-additional-output-formats.md) | Word & Excel Quote Export | |
| [T038](../tasks/T038-esignature-integration.md) | E-Signature Integration | |
| [T039](../tasks/T039-multitenant-hardening.md) | Multi-Tenant Hardening | |
| [T040](../tasks/T040-role-permission-system.md) | Role & Permission System | |
| [T041](../tasks/T041-hubspot-marketplace-prep.md) | HubSpot App Marketplace Prep | |
| [T042](../tasks/T042-whitelabel-foundations.md) | White-Label Branding Foundations | |
| [T043](../tasks/T043-documentation-onboarding.md) | Documentation & In-App Onboarding | |

## Definition of Done
- Quotes support 2–4 option groups; prospects can select and accept one option on the hosted page.
- Quotes export to .docx and .xlsx with accurate line item tables and totals.
- DocuSign and Adobe Sign envelopes are created, signed, and status-synced via webhook.
- Integration tests confirm no query can return data from a different portal_id.
- Rep, Manager, Admin, and Finance roles have correctly scoped data access.
- App passes HubSpot's pre-review checklist; rate limit handling and webhook reliability are verified.
- Portal branding (logo, primary color, font) is applied to generated quote output.
