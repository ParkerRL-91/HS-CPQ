# HubSpot CPQ Platform

A HubSpot-native Configure, Price, Quote (CPQ) platform designed to match enterprise CPQ depth (multi-dimensional pricing, discount waterfalls, approval chains, subscription management) while being faster to deploy and deeply integrated with HubSpot's CRM data.

## Project Structure

```
docs/
  sprints/      Sprint summaries and goals (sprint-01.md … sprint-10.md)
  tasks/        Individual task specs (T001 … T048)
```

## Sprints at a Glance

| Sprint | Phase | Weeks | Focus |
|--------|-------|-------|-------|
| [Sprint 01](docs/sprints/sprint-01.md) | 1 | 1–4 | Foundation: OAuth, DB, Product Sync, Basic Pricing |
| [Sprint 02](docs/sprints/sprint-02.md) | 1 | 5–8 | Core Quoting: Builder, PDF, Delivery, Sidebar |
| [Sprint 03](docs/sprints/sprint-03.md) | 2 | 9–12 | Advanced Pricing: MDQ, Discount Schedules, Rules, Bundles |
| [Sprint 04](docs/sprints/sprint-04.md) | 2 | 13–16 | Approvals, Multi-Currency, Sales Agreements |
| [Sprint 05](docs/sprints/sprint-05.md) | 3 | 17–20 | Subscriptions & Renewals |
| [Sprint 06](docs/sprints/sprint-06.md) | 3 | 21–24 | Dashboards, Versioning & Collaboration |
| [Sprint 07](docs/sprints/sprint-07.md) | 4 | 25–28 | Template Engine & Guided Selling |
| [Sprint 08](docs/sprints/sprint-08.md) | 4 | 29–32 | Commercial Readiness & Marketplace |
| [Sprint 09](docs/sprints/sprint-09.md) | 5 | 33–37 | Usage-Based Pricing & Agreement Workflows |
| [Sprint 10](docs/sprints/sprint-10.md) | 5 | 38–42 | AI Intelligence & Partner Portal |

## Tech Stack Summary

- **Frontend:** Next.js 14 (App Router), shadcn/ui + Tailwind, Zustand + React Query
- **Backend:** Next.js API Routes + tRPC, Prisma + PostgreSQL (Supabase/Neon)
- **HubSpot:** OAuth 2.0 Public App, UI Extensions, CRM API, Webhooks
- **Infrastructure:** Vercel → AWS, Clerk/Auth.js, Cloudflare R2/S3, Sentry + PostHog
