# T017 — Multi-Currency Support

**Sprint:** [Sprint 04](../sprints/sprint-04.md) | **Phase:** 2

## User Stories
- As an admin, I can configure the currencies my portal supports and set exchange rates.
- As a rep, I can select the currency when creating a quote and all prices, discounts, and totals display in that currency.
- As a manager, all quote analytics are reported in the portal's base currency regardless of the quote currency.

## Definition of Success
- Admin can add supported currencies (ISO code, symbol, display name) and configure exchange rate mode per currency: manual, daily-sync, or locked-at-creation.
- Daily-sync mode fetches from an FX rate API (e.g., Open Exchange Rates) via a scheduled Inngest job.
- At quote creation, the active exchange rate for the selected currency is stored on the `Quote` record (`fx_rate`).
- All line item prices are stored in both the quote currency and base currency equivalent.
- Approval threshold comparisons use the base currency equivalent.
- Analytics queries use the `base_currency_total` field for consistent cross-currency aggregation.

## Files & Objects Touched
- `packages/db/prisma/schema.prisma` — `Currency` model, `Quote.currency_code`, `Quote.fx_rate`, `Quote.base_currency_total`, `LineItem.base_currency_total`
- `apps/web/app/admin/currencies/page.tsx` — currency configuration
- `packages/jobs/src/fx-rate-sync.ts` — Inngest scheduled job
- `packages/pricing/src/currency.ts` — conversion utilities
- `packages/api/src/routers/currency.ts`

## Methodology
1. Define `Currency` schema: `portal_id`, `iso_code`, `symbol`, `name`, `rate_mode` (manual | daily_sync | locked), `manual_rate`, `last_synced_rate`, `last_synced_at`.
2. Admin currency page: add/remove currencies, set rate mode, manually enter rates for manual mode.
3. Implement `fx-rate-sync` Inngest job: daily at midnight UTC, fetches latest rates for all `daily_sync` currencies from the FX API; updates `last_synced_rate` and `last_synced_at`.
4. At quote creation, snapshot the active rate onto `Quote.fx_rate`; for `locked` mode, this rate is fixed for the quote's lifetime.
5. Implement `currency.ts` conversion: `toBaseCurrency(amount, fxRate)` and `fromBaseCurrency(amount, fxRate)` pure functions.
6. In the pricing engine, store both the quote-currency price and the base-currency equivalent on each line item; approval trigger comparisons always use base-currency totals.
