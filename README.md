# AmoraAsia

A serious-intent relationship platform for Southeast Asia. Production at
**[amoraasia.com](https://amoraasia.com)**.

The product bet is narrow and deliberate: people looking for a long-term
relationship are badly served by swipe queues optimised for session length.
AmoraAsia leads with a verified member directory that logged-out visitors can
browse, and gates the thing that actually needs an account — talking to
someone.

---

## Stack

| Layer | Choice |
|---|---|
| Framework | Next.js 16 (App Router, Turbopack), React 19 |
| Language | TypeScript, strict |
| Styling | Tailwind CSS v4 (`@theme inline`), design tokens in `app/globals.css` |
| Auth + data | Firebase Auth, Cloud Firestore (`firebase-admin` server-side only) |
| Media | Vercel Blob for member photos, `next/image` with explicit remote patterns |
| Billing | Stripe subscriptions — tier/interval ladder in `lib/tiers.ts` |
| Analytics | PostHog (EU ingestion), wired via `instrumentation-client.ts` |
| Validation | Zod at every API boundary |
| Hosting | Vercel |

## Layout

```
app/
  (legal)/          Terms, privacy, community guidelines
  api/              Route handlers — see below
  browse/           The member directory
  matches/  liked-me/           Mutual interest + inbound interest
  messages/ (via api/messages)  Gated conversation
  premium/          Subscription tiers and checkout entry
  profile/ settings/ sign-up/   Account lifecycle
components/         29 client components
config/             Brand + market configuration
lib/                Domain logic, kept framework-agnostic
docs/               Real setup runbooks, one per external service
```

### API surface

`account` · `apply` · `assetlinks` · `contact-exchange` · `health` ·
`liked-me` · `likes` · `messages` · `photos` · `showcase` · `stripe` ·
`verify-selfie` · `voice-intro`

`firebase-admin` is imported **only** inside `app/api/**` handlers, via
`lib/firebaseAdmin.ts`. Importing it into a `"use client"` file breaks the
build — it is Node-only.

## Notable engineering

- **Guest-browsable directory.** `/api/showcase` serves the public member
  grid with edge revalidation, returning first name, age and photo only —
  never the fields given in confidence. Members are included by default and
  can opt out, rather than the reverse.
- **Photo pipeline.** Upload → moderation → activation
  (`lib/moderation.ts`, `lib/photoActivation.ts`) before anything is
  publicly visible. Selfie verification is a separate route.
- **Tiered billing.** `lib/tiers.ts` is the single source of truth for the
  price ladder; `/api/stripe/price?verify=1` resolves every configured
  price ID against the live Stripe account, so a misconfigured interval
  surfaces as a failed check rather than a wrong charge.
- **Market configuration.** `lib/markets.ts` keeps city/market specifics out
  of components.

## Running it

```bash
npm install
npm run dev          # http://localhost:3000
npm run build        # production build
npm run lint
```

Every external service has its own runbook in [`docs/`](./docs) —
Stripe, Vercel Blob, PostHog, Google Vision, Groq, phone auth, push
notifications, and the Android build.

Environment variables are documented in `.env.example`. Note that
`NEXT_PUBLIC_*` values are **inlined at build time**: changing one in Vercel
requires a real rebuild, not a redeploy of the cached build.

## Conventions

`AGENTS.md` is the contributor guide and is binding for both humans and
coding agents. `CLAUDE.md` points at it. Read it before changing the data
model, the Firestore rules, or anything under `app/api/`.

---

© Kuchimoto Studio. Built and maintained by
[Samuel Louissaint](https://github.com/samkuchimoto).
