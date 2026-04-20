# Sprint 06 — Dashboards, Versioning & Collaboration

**Phase:** 3 | **Weeks:** 21–24

## Goal
Deliver the standard dashboard suite (quote pipeline, discount analysis, renewal health, approval activity, quote engagement), implement quote versioning with immutable snapshots, add quote engagement tracking on the hosted quote page, and enable internal collaboration via comment threads and review requests.

## Summary
This sprint gives managers visibility into the sales engine and gives reps the tools to collaborate on complex deals. Dashboards surface quote health, pricing trends, and renewal risk. Quote versioning creates an immutable paper trail so every approved/sent version is auditable. Engagement tracking on the hosted quote page reveals prospect behavior. Internal comment threads and review requests let deals be worked on collaboratively without losing audit context.

## Tasks
| ID | Task | Owner |
|----|------|-------|
| [T025](../tasks/T025-quote-versioning.md) | Quote Versioning | |
| [T026](../tasks/T026-quote-engagement-tracking.md) | Quote Engagement Tracking | |
| [T027](../tasks/T027-internal-collaboration.md) | Internal Quote Collaboration | |
| [T028](../tasks/T028-quote-pipeline-dashboard.md) | Quote Pipeline Dashboard | |
| [T029](../tasks/T029-discount-analysis-dashboard.md) | Discount Analysis Dashboard | |
| [T030](../tasks/T030-renewal-health-dashboard.md) | Renewal Health Dashboard | |
| [T031](../tasks/T031-approval-engagement-dashboards.md) | Approval Activity & Engagement Dashboards | |

## Definition of Done
- Auto-versioning creates a new immutable version when an approved/sent quote is edited.
- Version comparison renders a side-by-side diff; prior versions are restorable.
- Engagement events (first open, re-open, time-per-section, scroll depth) are tracked and stored.
- Comment threads, @mentions, and review requests work end-to-end with email notifications.
- All five standard dashboards render with real data and correct metric definitions.
