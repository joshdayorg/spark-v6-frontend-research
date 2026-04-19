# Spark v6 Frontend Research

Research, vision docs, and design references for the Spark v6 frontend.

## What's Here

### Research
- **[Chat 2.0: The Controller Paradigm](research/2026-04-18-chat-controller-research.md)** — Deep research into replacing the chat box with a structured Who/What/When/Where/Why controller. Covers prior art, GitHub repos, academic papers, design manifestos, technical architecture, and implementation plan.

## The Vision

Chat is a dead end for structured work. The future is **structured intent surfaces** — controls that decompose what you want into navigable dimensions, auto-complete your intent, and render the right UI for the response.

The five primitives: **Who, What, When, Where, Why.**

## Related Repos

- `spark-v6` — Specs, architecture, product vision
- `spark-v6-codex` — Convex backend (schema, functions, agents)
- `spark-v6-frontend-jd` — Frontend implementation

## Stack

- Next.js (App Router)
- Vercel AI SDK + AI Elements
- Convex (real-time backend)
- shadcn/ui + Tailwind CSS v4
- Clerk (auth)
