# T025 — Quote Versioning

**Sprint:** [Sprint 06](../sprints/sprint-06.md) | **Phase:** 3

## User Stories
- As an auditor, I can see every version of a quote ever approved or sent, with an immutable snapshot of line items and pricing at that moment.
- As a rep, I can compare any two versions side by side to see exactly what changed.
- As a rep, I can restore a prior version as the new working version if needed.

## Definition of Success
- A new version is auto-created whenever a quote in `approved` or `sent` status is edited.
- Reps can manually save a named version at any point ("v2 — after negotiation call").
- Side-by-side version comparison shows changed line items, pricing deltas, and changed quote terms.
- Any prior version can be restored; restoring triggers re-approval if the restored version differs from the last approved version.
- Each `ApprovalRequest` is linked to the specific `QuoteVersion` it was initiated against.

## Files & Objects Touched
- `packages/db/prisma/schema.prisma` — `QuoteVersion` model (snapshot JSON, version number, label, created_by)
- `packages/quotes/src/version-manager.ts` — snapshot creation and restore logic
- `apps/web/app/quotes/[quoteId]/versions/page.tsx` — version history list
- `apps/web/components/quotes/VersionComparison.tsx` — side-by-side diff view
- `packages/api/src/routers/quote.ts` — `saveVersion`, `restoreVersion`, `compareVersions` procedures

## Methodology
1. Define `QuoteVersion`: `quote_id`, `version_number` (auto-incrementing per quote), `label` (optional), `snapshot` (JSON containing full line item list with prices, quote terms, applied rules), `created_by_user_id`, `created_at`. Immutable after creation.
2. `version-manager.ts`: `createSnapshot(quoteId)` — serializes the current quote state into `snapshot` JSON and inserts a new `QuoteVersion`. Called automatically before any mutation when `quote.status in (approved, sent)`.
3. `restoreVersion(quoteVersionId)`: copies the version's snapshot into the live quote state; increments version number; if the restored state differs from the last approved version, sets `quote.status = draft` to trigger re-approval.
4. Version history page: chronological list of versions with number, label, created by, date, and links to view or restore. Current version highlighted.
5. `VersionComparison` component: side-by-side tables for two selected versions; changed rows highlighted in amber; added rows in green; removed rows in red. Summary header shows total value delta and changed field count.
6. Link `ApprovalRequest.quote_version_id` at the time of submission so approvers always see the version they approved.
