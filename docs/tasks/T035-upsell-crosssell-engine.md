# T035 — Upsell & Cross-Sell Recommendation Engine

**Sprint:** [Sprint 07](../sprints/sprint-07.md) | **Phase:** 4

## User Stories
- As a rep, I see relevant upsell and cross-sell suggestions in a side panel while building a quote.
- As a rep, each recommendation shows a reason why it was suggested so I can use it in my sales conversation.
- As an admin, I can configure manual recommendation rules ("if Product A is on the quote, suggest Product B").

## Definition of Success
- `RecommendationRule` table stores admin-configured rules: trigger product → recommended product → reason label.
- Historical pattern analysis identifies products frequently co-purchased with the products currently on the quote (based on won deals).
- Bundle completeness detection: if a partial bundle is on the quote, recommend completing it.
- Recommendations panel shows up to 6 suggestions with one-click "Add to Quote" and a dismissal option.
- Each recommendation records an `impression` and (if acted on) an `added` event for analytics.

## Files & Objects Touched
- `packages/db/prisma/schema.prisma` — `RecommendationRule`, `RecommendationEvent`
- `packages/recommendations/src/rule-based.ts` — evaluates `RecommendationRule` records
- `packages/recommendations/src/pattern-based.ts` — co-purchase frequency analysis
- `packages/recommendations/src/bundle-completeness.ts` — detects partial bundles
- `apps/web/components/quote-builder/RecommendationPanel.tsx`
- `packages/api/src/routers/recommendations.ts` — `getRecommendations` procedure

## Methodology
1. `rule-based.ts`: for each product on the current quote, query `RecommendationRule` records where `trigger_product_id` matches; return the `recommended_product_id` and `reason_label` for rules whose recommended product is not already on the quote.
2. `pattern-based.ts`: query historical won `Quote` records; for each product currently on the quote, find the top 5 products most frequently co-occurring in won quotes (excluding products already on the quote); rank by co-occurrence frequency.
3. `bundle-completeness.ts`: check if any products on the quote are components of a bundle whose parent is not on the quote; suggest adding the parent bundle with the reason "Complete the bundle for a better price".
4. `getRecommendations` procedure: aggregates results from all three sources, deduplicates by product ID, ranks (rule-based first, then bundle completeness, then pattern-based), and returns the top 6.
5. `RecommendationPanel` component: renders each recommendation as a card (product name, list price, reason chip, "Add" button, "✕" dismiss). On "Add", calls `quote.addLineItem`; records an `added` event. On "✕", records a `dismissed` event.
6. Admin `RecommendationRule` management: simple table with product selectors and reason label input; no branching logic (that's for the guided selling wizard).
