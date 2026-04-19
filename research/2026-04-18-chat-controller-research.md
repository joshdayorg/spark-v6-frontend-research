# Chat 2.0: The Controller Paradigm — Research Synthesis

> **Vision**: Replace the chat box with a structured controller built around **Who, What, When, Where, Why** primitives that auto-generate queries as users interact — a cockpit, not a confessional.

---

## The Thesis

The entire AI industry is converging on one conclusion: **the chat box is a dead end for structured work**. The blank text field puts all cognitive burden on the user. The future is structured intent surfaces — controls that decompose what you want into navigable dimensions, auto-complete your intent, and render the right UI for the response.

Your "Who, What, When, Where, Why" controller is the most radical and clear expression of this direction. Nobody has built it yet as a unified control surface. Here's the landscape.

---

## 1. Key Prior Art & References

### Academic / Research

| Source | What It Is | Why It Matters |
|--------|-----------|----------------|
| **[SALT-NLP GenUI](https://github.com/SALT-NLP/GenUI)** (TypeScript, 121★) | Academic framework where LLMs generate UIs instead of text responses. 72% improvement in human preference over chat. Uses finite state machines for interface specification. | Proves the paradigm works. Their intermediate representation (structured UI specs) maps cleanly to your controller concept. |
| **[Microsoft Promptions](https://github.com/microsoft/Promptions)** | Dynamic prompt refinement controls — AI generates contextual UI elements (sliders, toggles, radio buttons) based on user input. Options auto-update responses in real-time. | Closest existing implementation to your "controls that auto-generate queries" concept. The Option Module → Chat Module pipeline is directly applicable. |
| **[Generative Interfaces for Language Models](https://arxiv.org/html/2508.19227v2)** (arXiv paper) | Full academic treatment. Proposes structured interface-specific representations + iterative refinement. | The theoretical backbone. Formalizes why chat underperforms and what replaces it. |

### Industry Protocols

| Protocol | What It Does | Link |
|----------|-------------|------|
| **A2UI (Agent-to-UI)** — Google | Agents return declarative JSON component trees instead of text. Host renders approved widgets. JSONL-based progressive rendering. | [a2ui-blazor](https://github.com/23min/a2ui-blazor) (Blazor impl) |
| **AG-UI (Agent-User Interaction Protocol)** | Real-time bidirectional streaming between agents and frontends. State events, tool calls, lifecycle management. | [ag-ui-swift](https://github.com/paduh/ag-ui-swift) |
| **MCP-UI** (Anthropic extension) | Extends Model Context Protocol to serve rich UI components from agent tool calls. | Emerging — watch Anthropic docs |

### Products & Frameworks

| Product/Framework | What It Does | Key Insight |
|------------------|-------------|-------------|
| **[Thesys / Crayon](https://www.thesys.dev/)** | Open-source React SDK for generative UI. Register components, agent picks them. C1 API returns structured UI instead of text. 300+ teams in prod. | Two component types: *generative* (rendered once) and *interactable* (persistent across conversation) — maps to your controller surfaces. |
| **[Tambo](https://tambo.co)** | React SDK where developers register existing components. Agent orchestrates which to render. Supports MCP integration. | Component registration pattern useful for your Who/What/When/Where/Why widgets. |
| **Vercel AI SDK + AI Elements** | 40+ pre-built chat components + `useChat` hook with tool invocations that can render custom React components inline. | **Already in our stack.** The tool-invoked component pattern is how we wire the controller to streaming AI responses. |
| **Nexla Express** | AI agent that generates XML UI specs (forms, OAuth flows, data explorers) inline in chat. Agent constructs the UI dynamically. | Proved that XML-defined component schemas in the system prompt let the agent generate rich UI on the fly. |

### Key Writings / Manifestos

| Article | Author | Core Argument |
|---------|--------|---------------|
| **["Beyond the Chatbox: Generative UI"](https://blog.devwithawais.com/beyond-the-chatbox-how-generative-ui-is-rescuing-us-from-the-wall-of-text-35e94b89b5f1)** | Muhammad Awais (Apr 2026) | Three-dimensional matrix: Workflow Complexity × AI Autonomy × AI Reasoning determines which UI modality to surface (prompt bar → contextual tooltip → immersive canvas). |
| **["From Screens to Cockpits"](https://medium.com/@aiforhuman/from-screens-to-cockpits-the-new-form-factor-for-agentic-applications-fef49e9dd357)** | AI4HUMAN (Jan 2026) | Aviation cockpit analogy. Agentic apps need: intent visible before execution, authority explicit at action time, friction proportional to blast radius. **Pages → Timelines. Fields → Scopes. Buttons → Permissions.** |
| **["The End of Chat: Polymorphic Interfaces"](https://dev.to/mosiddi/the-end-of-chat-why-ai-interfaces-must-be-polymorphic-4mbn)** | mosiddi (Jan 2026) | Agent generates Intent + Data. Interface layer renders based on context. Output Modality Detector chooses: chart, table, ghost text, notification, dashboard widget. |
| **["Beyond Chat: The Interface Revolution"](https://www.mmntm.net/articles/beyond-chat-interfaces)** | MMNTM Research (Dec 2025) | Data: GUIs outperform chat for structured tasks. Three patterns: Tool-Invoked Components, Generative Design (v0), Sidecar Pattern (Artifacts). Ghost State design pattern (ambient copilot). |
| **["Adaptive UI: The Missing Layer"](https://marioottmann.com/articles/adaptive-ui-agentic-ai)** | Mario Ottmann (Mar 2026) | Three layers that stack: (1) Predefined components agent selects, (2) Agent-generated custom UI, (3) Cross-platform declarative protocol (A2UI). |
| **["Copilot as the UI"](https://figr.design/blog/copilot-as-the-ui)** | Figr (Mar 2026) | Evolution: Traditional UI → Conversational UI → Generative UI → Agentic UX. "Designers increasingly craft semantic architectures, APIs, schemas, not pixels." |
| **["AI Interface: When Intelligence Outgrows Its Container"](https://uxdesign.cc/ai-interface-when-intelligence-outgrows-its-container-78f7ddfa3341)** | Sen Lin / UX Collective | Category error: AI products designed as "windows" but capable as "rooms." Power users need control panels, not chat. Dashboard > chat box for complex work. |
| **["Beyond Chatbots: AI Interface Design That Drives Adoption"](https://caylent.com/blog/beyond-chatbots-ai-interface-design-that-drives-adoption)** | Caylent (Feb 2026) | 5 alternatives that outperform chatbots: Predictive Dashboards, Draft-and-Refine, Embedded Nudges, Dynamic Forms with Contextual Assistance, Workflow Automation. |

---

## 2. The "Who, What, When, Where, Why" Controller — Design Concept

### Why These Five Dimensions?

Every query a human has can be decomposed along these five axes. They're the fundamental dimensions of intent:

| Dimension | Controls | Auto-generates |
|-----------|----------|----------------|
| **WHO** | People picker, entity selector, @mentions, role/relationship filter | Queries scoped to people, teams, contacts, agents |
| **WHAT** | Topic/action selector, verb palette, content type filter, goal picker | The core action/content of the query |
| **WHEN** | Timeline scrubber, date range, recency filter, temporal context | Time-scoped queries, scheduling, historical analysis |
| **WHERE** | Source picker, channel selector, location/context, data source | Queries routed to specific systems, repos, files, locations |
| **WHY** | Intent/goal selector, reasoning depth, output format, purpose | Shapes how the AI interprets and responds — analysis vs. summary vs. action |

### How It Works (Interaction Flow)

```
┌─────────────────────────────────────────────────────┐
│                  SPARK CONTROLLER                     │
│                                                       │
│  ┌─────┐  ┌──────┐  ┌──────┐  ┌───────┐  ┌─────┐  │
│  │ WHO │  │ WHAT │  │ WHEN │  │ WHERE │  │ WHY │  │
│  └──┬──┘  └──┬───┘  └──┬───┘  └──┬────┘  └──┬──┘  │
│     │        │         │         │           │      │
│     ▼        ▼         ▼         ▼           ▼      │
│  ┌─────────────────────────────────────────────┐    │
│  │        LIVE QUERY PREVIEW (auto-updating)    │    │
│  │  "Show me [WHO's] [WHAT] from [WHEN]         │    │
│  │   in [WHERE] for [WHY purpose]"              │    │
│  └─────────────────────────────────────────────┘    │
│                        │                             │
│                        ▼                             │
│  ┌─────────────────────────────────────────────┐    │
│  │        RESPONSE SURFACE (generative UI)      │    │
│  │  Renders: cards, charts, tables, timelines,  │    │
│  │  diffs, forms — whatever fits the query       │    │
│  └─────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────┘
```

### Key Behaviors

1. **Auto-complete as you type** — like Google's predictive search, but across all 5 dimensions simultaneously
2. **Dimension controls are contextual** — selecting a WHO narrows WHAT options, selecting WHEN filters WHERE suggestions
3. **Live query synthesis** — as you manipulate any control, a natural language query is assembled and previewed in real-time
4. **The AI also suggests** — based on partial input, the controller proposes complete queries ("Did you mean...?")
5. **Response adapts to dimensions** — WHO-heavy queries render people cards, WHEN-heavy queries render timelines, WHAT-heavy queries render content blocks
6. **Chat is still there** — but as a fallback/power-user input, not the primary surface

---

## 3. Technical Architecture for Spark

### Stack Alignment

Our existing stack is built for this:

- **Vercel AI SDK** `useChat` + tool invocations → wire each dimension to AI-powered autocomplete
- **AI Elements** → pre-built components for rendering responses (messages, code blocks, reasoning)
- **Convex real-time subscriptions** → live query preview updates as user manipulates controls
- **shadcn/ui** → base layer for the controller chrome (command palette, popovers, selectors)
- **Next.js App Router** → server components for the controller layout, client components for interactive dimensions

### Proposed Component Architecture

```
src/
  components/
    controller/
      SparkController.tsx          # Main 5W controller layout
      DimensionBar.tsx             # Horizontal bar of 5 dimension controls
      dimensions/
        WhoDimension.tsx           # People/entity picker with AI autocomplete
        WhatDimension.tsx          # Action/topic selector with verb palette
        WhenDimension.tsx          # Timeline scrubber + date controls
        WhereDimension.tsx         # Source/context picker
        WhyDimension.tsx           # Intent/purpose selector + output format
      QueryPreview.tsx             # Live natural-language query synthesis
      QuerySuggestions.tsx         # AI-generated query completions
      ResponseSurface.tsx          # Generative UI response renderer
    chat/
      ChatFallback.tsx             # Traditional chat as secondary input
```

### Implementation Approach (Promptions-inspired)

Borrow Microsoft's Promptions architecture:

1. **Option Module** — Each W dimension generates structured options based on context
2. **Synthesis Module** — Combines dimension selections into a coherent query
3. **Response Module** — Generates response using the structured query (not raw text)
4. **Render Module** — Picks the right UI components based on query type

---

## 4. Design References & Inspiration

### Direct Inspirations
- **Microsoft Promptions** — dynamic controls generated from user input
- **Zeta Alpha** — neural search with structured controls, faceted search
- **Figma's AI** — spatial controls, not chat
- **Linear's Triage Intelligence** — structured input with keyboard shortcuts, pre-filled metadata
- **Cursor's Tab/Ghost Text** — ambient AI, zero-click acceptance
- **Google Search AI Mode** — query → custom UI generation

### Cockpit / HUD References
- Aviation cockpit metaphor (AI4HUMAN article) — expose the right abstraction at the right level
- Gaming HUDs — structured overlays that provide context without obscuring the workspace
- Bloomberg Terminal — the ultimate structured query interface, dimensions everywhere

---

## 5. Implementation Plan for Spark v6 Frontend

### Phase 1: Controller Chrome (Week 1)
- Build `SparkController` layout with 5 dimension slots
- Use shadcn Command palette / Popover for each dimension
- Wire basic dimension state management (Zustand or React context)
- Design the dimension bar (horizontal strip, expandable panels)

### Phase 2: AI-Powered Dimensions (Week 2)
- Wire each dimension to Convex-backed AI autocomplete
- Implement `QueryPreview` that synthesizes dimensions into natural language
- Add `QuerySuggestions` — AI proposes complete queries from partial input
- Cross-dimension filtering (selecting WHO narrows WHAT suggestions)

### Phase 3: Generative Response Surface (Week 3)
- Build `ResponseSurface` using AI Elements + custom widgets
- Map query types to response renderers (people → cards, time → timeline, etc.)
- Integrate with existing Convex Agent threads for AI execution
- Stream responses via `useChat` with tool-invoked components

### Phase 4: Polish & Chat Fusion (Week 4)
- Add keyboard shortcuts for power users (Cmd+1 through Cmd+5 for dimensions)
- Implement query history (bottom rail, structured thread)
- Merge chat as a secondary input that auto-populates dimensions
- Animation, transitions, the "from the future" feel

---

## 6. Open Questions

1. **Mobile layout** — How do 5 dimensions work on a phone? Swipeable carousel? Bottom sheet with tabs?
2. **Default state** — What does the controller show before any input? Suggested queries? Recent activity?
3. **Hybrid mode** — Can a user type freely and have dimensions auto-populate (reverse direction)?
4. **Persistence** — Do dimension selections persist across sessions? (Convex makes this easy)
5. **Agent threads** — Does each controller query create a new thread, or extend the current one?

---

*Research compiled 2026-04-18. Sources span Dec 2024 – Apr 2026.*
