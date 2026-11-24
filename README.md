# Subscription Billing Platform

> Case study: a premium subscription platform with recurring billing, tiered content gating across text/audio/video, community features, and a full admin analytics dashboard.
> Built for **Nairametrics — Nigeria's largest business newsroom**.
> **Code is proprietary; this repo documents the build** — architecture, decisions, and outcomes.

---

## Context

| | |
|---|---|
| **Client** | Nairametrics (FTM — Follow The Money, premium stock-recommendation newsletter) |
| **Role** | Full-stack lead — platform architecture, payment integration, API design |
| **Timeline** | Oct 2024 — present (ongoing platform) |
| **Engagement** | Lead Product Engineer, Nairametrics |

## Problem

Nairametrics wanted to move its financial newsletter from free content to a gated, paid model. The platform had to handle the full subscription lifecycle — not one-off payments — and gate content consistently across text, audio, and video, while giving the newsroom's commercial team a dashboard they could actually run the business from: subscribers, revenue, churn, and payment failures.

## Architecture

```
┌─────────────────────────┐        ┌─────────────────────────┐
│  Next.js 15 (App Router)│  REST  │  Laravel 10 (PHP 8.2+)  │
│  TypeScript, Tailwind 4 │ ◀────▶ │  PostgreSQL/MySQL, Redis│
│  Content + community UI │  JWT   │  Paystack webhooks      │
└─────────────────────────┘        └───────────┬─────────────┘
                                               │
                                     ┌─────────▼─────────┐
                                     │  Admin dashboard  │
                                     │  MRR, churn, KPIs │
                                     └───────────────────┘
```

### Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 15 (App Router), TypeScript 5.7, Tailwind CSS 4, Redux Toolkit, React Hook Form |
| Content | SunEditor/Draft.js rich text, audio/video playback, Recharts for stock-pick data visualization |
| Backend | Laravel 10, PHP 8.2+, PostgreSQL/MySQL, Redis caching |
| Auth | JWT access + refresh tokens, CSRF protection |
| Payments | Paystack (recurring subscriptions, webhooks) |
| Exports | Laravel Excel (subscribers, financial data), FPDF/FPDI (reports, invoices) |

### Subscription billing (Paystack)

The core of the platform is the billing lifecycle: recurring subscriptions rather than one-off payments, with idempotent webhook handling, edge-case coverage, and retry logic so a missed or duplicated webhook never corrupts subscription state. The admin dashboard exposes revenue tracking, churn metrics, and the subscriber lifecycle.

### Content gating

Tiered access boundaries apply uniformly across media types — text articles, audio podcasts, and video. Public teasers and structured data preserve SEO visibility for a newsroom whose traffic is the top of the commercial funnel. The editorial workflow supports rich-text publishing (SunEditor/Draft.js), media uploads, and scheduled publishing from the admin dashboard — the commercial team runs the content operation without engineering involvement.

### Community

Threaded comments with a moderation workflow and spam detection (automated plus manual review queues), user profiles with subscription status, comment history, and saved content.

### Admin dashboard

User management (roles, subscription status, support actions), content management (publishing, media uploads, scheduling), payment analytics (MRR, churn, LTV, payment failure tracking), and a real-time KPI view.

### API

RESTful API with standardized error responses across 40+ endpoints and **1,617 lines of maintained API documentation** — kept current as endpoints evolve.

### Operations & roadmap (as of this writing)

An honest snapshot of where the platform's engineering stands: a marketing/transactional email migration (Mailchimp) is queued, test-coverage targets (Jest/React Testing Library on the frontend, PHPUnit on the backend) are scheduled work, and the Paystack webhook handling gets recurring edge-case reviews as real payment traffic surfaces new scenarios.

## Key Technical Decisions

| Decision | Rationale | Trade-off |
|---|---|---|
| Paystack over Stripe | Nigerian market: local card support, NGN settlement, lower fees | International expansion will need Stripe added alongside |
| JWT (access + refresh) over session auth | Stateless API, mobile-ready, microservice-compatible | Token refresh and revocation complexity |
| Laravel for the backend | Team expertise, mature ecosystem, rapid delivery for a business-critical billing system | Heavier than Node.js would be for any future real-time features |
| Redis caching layer | Newsroom traffic spikes (market-moving days) hit the same hot content repeatedly | Another infrastructure component to operate |
| Uniform gating across media types | One authorization model, enforced server-side, regardless of how content renders | Media-specific edge cases (e.g., streaming range requests) need explicit handling |

## Outcomes

- **Client-reported**: the platform runs the full paid-content operation, with billing, gating, community and commercial analytics in one system.
- 1,617 lines of API documentation and standardized error responses mean new contributors onboard against a written contract, not tribal knowledge.

## What I'd Do Differently

- **Test coverage earlier.** Frontend (Jest/RTL) and backend (PHPUnit) coverage targets were set late; regressions in billing flows are the most expensive kind to catch in production.
- **Email service from day one.** The marketing/transactional email migration was deferred and became recurring technical debt — billing events deserve first-class email infrastructure immediately.
- **Subscription pause/resume.** Users asked for a pause option after launch; designing lifecycle states beyond active/cancelled up front would have made this a config change instead of a feature.

## Evidence

- **Live platform:** available on request.
- **API documentation:** available on request.
- **Screenshots:** available on request.

---

*Case study by David Aloba. [mistertaavetti.davidaloba.com](https://mistertaavetti.davidaloba.com) · [LinkedIn](https://linkedin.com/in/mistertaavetti)*
