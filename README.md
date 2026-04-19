# Spark v6 Frontend Research

Research, vision docs, and design references for the Spark v6 frontend.

## What's Here

### Plans
- **[Cockpit Page — Master Plan](plans/cockpit-page-master-plan.md)** — The full spec for the Cockpit homepage. Six-dimension controller (Who/What/When/Where/Why/How), ReUI + OpenUI integration, component architecture, interaction flows, and phased build plan.

### Research
- **[Chat 2.0: The Controller Paradigm](research/2026-04-18-chat-controller-research.md)** — Deep research into replacing the chat box with a structured controller. Covers prior art, GitHub repos, academic papers, design manifestos, technical architecture.

### Articles
- **[From Screens to Cockpits](articles/from-screens-to-cockpits.md)** — The aviation cockpit analogy for agentic application interfaces.

## The Vision

Spark has two interaction modes:

- **Cockpit** — Structured intent controller. Six dimension pills (Who/What/When/Where/Why/How). Auto-generating queries. Instrument-panel responses. The homepage.
- **Chat** — Traditional conversational AI. Text in, text out. The fallback for freeform conversation.

Cockpit is the primary surface. Chat is the escape hatch.

## Stack

- Next.js 16 (App Router)
- Vercel AI SDK + AI Elements
- **OpenUI** (generative UI protocol — LLM composes your components)
- **ReUI** (dashboard-grade components on shadcn)
- shadcn/ui + Tailwind CSS v4
- Convex (real-time backend)
- Clerk (auth)

## Related Repos

- `spark-v6` — Specs, architecture, product vision
- `spark-v6-codex` — Convex backend (schema, functions, agents)
- `spark-v6-frontend-jd` — Frontend implementation
