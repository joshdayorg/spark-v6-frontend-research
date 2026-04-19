# Spark v6 Frontend Research

Research, vision docs, and design references for the Spark v6 frontend.

## What's Here

### Plans
- **[⭐ v3: Chat IS the Cockpit](plans/v3-unified-cockpit-chat.md)** — CURRENT PLAN. The home screen is both cockpit and chat. 5W1H dimensions show live intelligence AND serve as input controls. The business is a sentient being communicating its state.
- [Cockpit Page Master Plan (v2)](plans/cockpit-page-master-plan.md) — Superseded. Separate cockpit page approach.
- [Step-by-Step Build Plan (v2)](plans/step-by-step-build-plan.md) — Superseded. Based on separate page approach.
- [Build Plan Audit](plans/build-plan-audit.md) — Audit of v2 plan. Most routing issues eliminated by v3.

### Research
- **[Chat 2.0: The Controller Paradigm](research/2026-04-18-chat-controller-research.md)** — Deep research into structured intent interfaces. Prior art, repos, papers, manifestos.

### Articles
- **[From Screens to Cockpits](articles/from-screens-to-cockpits.md)** — The aviation cockpit analogy for agentic interfaces.

## The Vision

The home screen IS the cockpit. Six dimensions of intelligence: **Who, What, When, Where, Why, How.** Each dimension shows the business communicating its own state — and each is a search/query entry point.

The business is a sentient being. It knows its people, its work, its timeline, its systems, its purpose, and its processes. The home screen is how it talks to you.

## Stack

- Next.js 16 (App Router)
- Vercel AI SDK + AI Elements
- **OpenUI** (generative UI — powers vitals + response widgets)
- shadcn/ui + Tailwind CSS v4
- Convex (real-time backend)
- Clerk (auth)
