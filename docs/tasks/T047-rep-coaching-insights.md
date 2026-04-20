# T047 — Rep Coaching Insights

**Sprint:** [Sprint 10](../sprints/sprint-10.md) | **Phase:** 5

## User Stories
- As a manager, I can see each rep's discount patterns and win rate correlation so I know who to coach on pricing discipline.
- As a manager, I receive a weekly coaching digest that surfaces the reps most at risk of pricing-related losses.
- As a rep, I can see my own performance benchmarks compared to my peers (anonymized) to understand how I'm doing.

## Definition of Success
- Coaching dashboard shows per-rep: average discount %, win rate by discount band, approval request frequency, and floor price breach rate.
- "Coaching flags" are automatically generated for reps who exceed thresholds (e.g., average discount > 20% + win rate < 50%).
- Weekly manager digest email surfaces the top 3 coaching opportunities from the rep's team with specific metrics.
- Reps see their own metrics on a personal performance page with anonymized peer benchmarks.
- All data is scoped to the manager's direct reports hierarchy.

## Files & Objects Touched
- `packages/analytics/src/rep-coaching.ts` — coaching metrics computation
- `packages/jobs/src/manager-coaching-digest.ts` — weekly digest job
- `apps/web/app/dashboard/coaching/page.tsx` — manager coaching dashboard
- `apps/web/app/dashboard/my-performance/page.tsx` — rep self-service page
- `packages/email/src/templates/coaching-digest.tsx`
- `packages/db/src/queries/rep-performance.ts`

## Methodology
1. Define coaching metrics per rep: average discount %, win rate (accepted / (accepted + expired) by discount band), approval trigger frequency (approval requests / quotes submitted), floor price breach rate, time-to-close average.
2. "Discount band" win rate: bucket quotes by discount % (0–10%, 10–20%, 20–30%, 30%+); compute win rate per band per rep; surface bands where win rate drops disproportionately as the band increases — this is the "diminishing returns" analysis.
3. Coaching flag generation: configurable thresholds (admin-set); when a rep's 30-day metrics cross a threshold combination, create a `CoachingFlag` record for the manager.
4. Manager coaching dashboard: table of direct reports with all metrics; coaching flags highlighted; click-through to individual rep's detailed breakdown.
5. Weekly digest job: for each manager, retrieve direct reports' flags from the past week; rank by ARR impact; render top 3 in the email with specific recommendation ("Rep X's win rate drops by 40% when discount exceeds 20% — discuss pricing confidence").
6. Rep self-service page: shows own metrics with anonymized percentile position among peers ("You are in the top 30% for average discount discipline"). No names or individual peer data exposed.
