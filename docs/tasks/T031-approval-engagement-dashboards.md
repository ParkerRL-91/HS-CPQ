# T031 — Approval Activity & Engagement Dashboards

**Sprint:** [Sprint 06](../sprints/sprint-06.md) | **Phase:** 3

## User Stories
- As a VP, I can identify approval bottlenecks — which approvers have the highest average cycle time — so I can address process issues.
- As an admin, I can see the most common rejection reasons so I can tune approval thresholds and training.
- As a manager, I can see quote engagement analytics (first open rate, time-to-accept) to evaluate quote quality.

## Definition of Success
- Approval activity dashboard: pending approvals by approver with SLA countdown, cycle time by trigger type, rejection rate and top rejection reasons, bottleneck analysis (average days spent at each approver).
- Quote engagement dashboard: first-open rate (% of sent quotes opened within 24hrs), average time from sent to accepted, section dwell time heatmap, re-open pattern analysis.
- Both dashboards share the same filter bar as the pipeline dashboard.

## Files & Objects Touched
- `apps/web/app/dashboard/approvals/page.tsx`
- `apps/web/app/dashboard/engagement/page.tsx`
- `apps/web/components/dashboard/ApproverSLATable.tsx`
- `apps/web/components/dashboard/RejectionReasonsChart.tsx`
- `apps/web/components/dashboard/EngagementHeatmap.tsx`
- `packages/db/src/queries/approvals.ts`
- `packages/db/src/queries/engagement.ts`

## Methodology
1. Approval cycle time query: `AVG(ApprovalStep.acted_at - ApprovalStep.activated_at)` grouped by `approver_user_id` and `trigger_type`.
2. SLA compliance table: pending `ApprovalStep` records with `activated_at` + SLA duration vs. current time; show hours remaining (or hours overdue in red).
3. Rejection reasons: group `ApprovalStep.comment` via keyword extraction (simple word-frequency analysis) or pre-defined reason categories that admins pick from when rejecting; count by reason.
4. Bottleneck analysis: for sequential chains, calculate average time spent at each step position; identify which position (step 1, 2, 3…) has the highest average dwell time.
5. Engagement first-open rate query: count quotes where `QuoteEngagementEvent.event_type = first_open AND created_at <= quote.sent_at + 24h` divided by total sent quotes.
6. Section dwell time heatmap: aggregate `section_view` events by `section_id`, average `duration_seconds`; render as a heatmap where sections are rows and time intensity is color-coded.
