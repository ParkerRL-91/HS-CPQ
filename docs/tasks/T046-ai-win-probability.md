# T046 — AI Win Probability Scoring

**Sprint:** [Sprint 10](../sprints/sprint-10.md) | **Phase:** 5

## User Stories
- As a rep, I see a win probability score on my quote so I know how likely the deal is to close.
- As a manager, I can see win probability scores across my team's pipeline to prioritize coaching and resources.
- As the system, the model is retrained periodically as new outcome data accumulates.

## Definition of Success
- Initial model trained on accumulated quote + outcome data (features: discount %, deal size, time in stage, product mix, rep tenure, company segment, engagement signals).
- Win probability score (0–100%) displayed on the quote detail page and in the manager pipeline dashboard.
- Model inference runs server-side; score is stored on `Quote.win_probability_score` and refreshed on each significant quote change.
- Model is retrained on a weekly schedule when new outcome data is available; new model version is staged and validated before replacing the production model.
- Model accuracy (AUC-ROC) is tracked over time in a model performance dashboard accessible to admins.

## Files & Objects Touched
- `packages/ml/src/win-probability/feature-extractor.ts` — extracts feature vector from a quote
- `packages/ml/src/win-probability/model.ts` — model inference (initially logistic regression or gradient boosting)
- `packages/ml/src/win-probability/trainer.ts` — training pipeline script
- `packages/db/prisma/schema.prisma` — `Quote.win_probability_score`, `ModelVersion`
- `packages/jobs/src/score-quote.ts` — Inngest job to score a quote
- `packages/jobs/src/retrain-model.ts` — weekly retraining job
- `apps/web/components/quotes/WinProbabilityBadge.tsx`

## Methodology
1. Define feature vector: normalize continuous features (discount %, deal size) to [0,1]; encode categorical features (company segment, product family) as one-hot vectors; include engagement signals (first-open rate, re-open count) and approval cycle time as features.
2. Implement `feature-extractor.ts`: given a `quoteId`, fetches all required data and returns a typed feature vector.
3. Initial model: use a gradient-boosted tree (via `ml-js` or a Python sidecar via HTTP API); train on `Quote` records with `status in (accepted, expired)` using `accepted` as the positive class.
4. `score-quote` Inngest job: triggered on quote status changes; runs inference, stores score on `Quote.win_probability_score`.
5. `retrain-model` weekly job: exports training data (feature vectors + outcomes), trains a new model, evaluates AUC-ROC against a held-out validation set, promotes the new model if AUC > current model's AUC.
6. `WinProbabilityBadge`: displayed on quote header and manager dashboard; color-coded (green > 70%, amber 40–70%, red < 40%); tooltip shows top contributing features.
