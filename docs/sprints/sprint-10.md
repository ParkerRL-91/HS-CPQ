# Sprint 10 — AI Intelligence & Partner Portal

**Phase:** 5 | **Weeks:** 38–42

## Goal
Train and deploy an AI win probability model that surfaces deal scores in the sidebar and manager dashboard. Build rep coaching insights that identify individual discount patterns and win rate correlations. Ship a partner portal MVP enabling channel partners to self-service quote within scoped product/discount access.

## Summary
The CPQ platform accumulates quote + outcome data across every deal. This sprint uses that data to build the first AI layer: a win probability model trained on deal attributes, pricing patterns, and acceptance outcomes. Rep coaching insights use the same data to surface coaching opportunities to managers. The partner portal gives channel partners a scoped, branded quoting experience — authenticated, limited to their assigned products and discount tiers — without exposing internal pricing logic.

## Tasks
| ID | Task | Owner |
|----|------|-------|
| [T046](../tasks/T046-ai-win-probability.md) | AI Win Probability Scoring | |
| [T047](../tasks/T047-rep-coaching-insights.md) | Rep Coaching Insights | |
| [T048](../tasks/T048-partner-portal.md) | Partner Portal MVP | |

## Definition of Done
- Win probability score is displayed in the HubSpot sidebar and manager dashboard per quote/deal.
- Model is retrained on a scheduled basis as new outcome data accumulates.
- Rep coaching dashboard shows individual discount patterns, win rate by discount band, and coaching flags for managers.
- Partner portal allows authenticated partner users to create quotes scoped to their assigned products and discount tiers.
- Partner quotes flow through the same approval engine with partner-specific approval chains.
