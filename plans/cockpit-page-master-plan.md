# Cockpit Page — Master Plan v2

> The homepage of Spark. Not a chat. A supercomputer interface built around **Who, What, When, Where, Why, How** — six dimensions of intent that replace the blank text box with a structured control surface.

---

## The Vision

Spark has two interaction modes:

| Mode | What it is | Entry point |
|------|-----------|-------------|
| **Cockpit** (NEW) | Structured intent controller. Six dimension pills. Auto-generating queries. Instrument-panel responses. The homepage. | `/` or `/cockpit` |
| **Chat** (EXISTING) | Traditional conversational AI thread. Text in, text out. Still valuable for open-ended, freeform conversation. | `/chat` or `/chat/[threadId]` |

They're separate but connected. You can start in Cockpit and a complex interaction might open a chat thread. Or you can go straight to chat. **Cockpit is the primary surface. Chat is the fallback.**

---

## Stack

### Already Have (keep everything)
| Tool | What it does | Status |
|------|-------------|--------|
| **Next.js 16** App Router | Page routing, server components | ✅ |
| **Vercel AI SDK** (`ai` + `@ai-sdk/react`) | `useChat`, `streamText`, model routing, tool execution | ✅ |
| **AI Elements** | Conversation components (for chat escalation) | ✅ |
| **shadcn/ui** | Base primitives (buttons, popovers, command palette, cards) | ✅ |
| **Tailwind CSS v4** | Styling | ✅ |
| **Recharts** | Charts | ✅ |
| **TanStack Table** | Data tables / grids | ✅ |
| **Convex** | Real-time backend, agent threads, subscriptions | ✅ |
| **Motion** (framer-motion) | Animations | ✅ |
| **cmdk** | Command palette | ✅ |

### Adding (one new dep)
| Tool | What it does | Why |
|------|-------------|-----|
| **[OpenUI](https://github.com/thesysdev/openui)** `@openuidev/react-lang` | Generative UI protocol. Define components with Zod → auto-generate system prompt → LLM responds in OpenUI Lang → streaming renderer displays your React components. 67% fewer tokens than JSON. | This is the core innovation. Lets the LLM compose your UI dynamically instead of returning text. |

### Build ourselves (with existing deps)
| Widget | Built with | When |
|--------|-----------|------|
| Timeline | shadcn + Tailwind | Phase 3 |
| Tree view | shadcn + recursive components | Phase 3 |
| Kanban | shadcn + @dnd-kit (small dep) | Phase 3 if needed |
| Filters / faceted search | shadcn popovers + badges | Phase 2 |
| Stepper | shadcn + Tailwind | Phase 3 |

### Dropped
| Tool | Why dropped |
|------|------------|
| ~~ReUI~~ | Not a library — it's a gallery of 1000+ shadcn example variations. Everything it offers can be built with what we already have. |

---

## Page Anatomy

```
┌─────────────────────────────────────────────────────────────────┐
│  SPARK COCKPIT                                    [⚙] [Chat →] │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    DIMENSION BAR                            ││
│  │                                                             ││
│  │  ┌─────┐  ┌──────┐  ┌──────┐  ┌───────┐  ┌─────┐  ┌─────┐││
│  │  │ WHO │  │ WHAT │  │ WHEN │  │ WHERE │  │ WHY │  │ HOW │││
│  │  └─────┘  └──────┘  └──────┘  └───────┘  └─────┘  └─────┘││
│  │                                                             ││
│  │  Small pills / capsules. Tap to expand. Glow when active.  ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                  QUERY SYNTHESIS BAR                        ││
│  │                                                             ││
│  │  "Show me [Sarah's] [deployment history] from [last        ││
│  │   week] in [GitHub] to [debug the outage]"                 ││
│  │                                                             ││
│  │  Live-updating as you interact with dimension pills.       ││
│  │  Also typeable — free text auto-populates dimensions.      ││
│  │                                       [Execute ⚡]         ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                  RESPONSE SURFACE                           ││
│  │                                                             ││
│  │  Generative — LLM picks & composes components via OpenUI.  ││
│  │  Streams in real-time as response arrives.                  ││
│  │                                                             ││
│  │  Components: Data tables, charts, timelines, cards,        ││
│  │  trees, steppers — all built with shadcn + existing deps.  ││
│  │  Registered as OpenUI library for LLM to compose.          ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                  SUGGESTION RAIL                            ││
│  │  [Related query 1]  [Related query 2]  [Dig deeper →]      ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

---

## The Six Dimensions

Each dimension is a **pill** — a small capsule button in the dimension bar. Tapping it opens an expanded panel/popover with AI-powered controls.

| Dimension | Pill Label | Expanded Controls | Example Selections |
|-----------|-----------|-------------------|--------------------|
| **WHO** | 👤 Who | People picker, @mentions, entity search, role filter | "Sarah Chen", "Backend Team", "Customer: Acme" |
| **WHAT** | 🎯 What | Action/topic selector, verb palette, content type | "deployments", "pull requests", "errors", "summarize" |
| **WHEN** | 🕐 When | Timeline scrubber, date range picker, recency presets | "Last 7 days", "Since v2.3 release", "Yesterday" |
| **WHERE** | 📍 Where | Source picker, repo/channel/system selector | "GitHub", "Slack #engineering", "Production logs" |
| **WHY** | 💡 Why | Goal/intent selector, reasoning depth, purpose | "Debug outage", "Prepare standup", "Audit security" |
| **HOW** | ⚙️ How | Output format, detail level, response style | "Dashboard view", "Brief summary", "Detailed report" |

### Pill States

- **Empty** — subtle outline, muted icon, no selection
- **Active** — expanded popover is open, pill glows/highlights
- **Filled** — dimension has a selection, pill shows compact summary ("👤 Sarah", "🕐 Last 7d")
- **AI-suggested** — subtle pulse/shimmer when AI recommends filling this dimension

### Cross-dimension intelligence

- Selecting WHO → narrows WHAT suggestions
- Selecting WHEN → filters WHERE options
- Selecting WHY → shapes HOW defaults
- AI proposes complete queries from partial input

---

## OpenUI Integration

### How it works

1. Build response widgets as normal React components (shadcn + Recharts + TanStack Table)
2. Wrap each with `defineComponent` + Zod schema via `@openuidev/react-lang`
3. Register them in a `cockpitLibrary`
4. OpenUI auto-generates a system prompt teaching the LLM what components exist
5. User manipulates dimensions → structured query sent to LLM with cockpit system prompt
6. LLM responds in OpenUI Lang (compact, streaming-first)
7. OpenUI's `<Renderer>` streams your components to screen as tokens arrive

### Component Library Definition

```typescript
// src/components/cockpit/openui/cockpit-library.ts
import { z } from "zod";
import { defineComponent, createLibrary } from "@openuidev/react-lang";

const DataTable = defineComponent({
  name: "DataTable",
  description: "Rich data table with sorting, filtering, pagination",
  props: z.object({
    title: z.string().optional(),
    columns: z.array(z.object({ key: z.string(), label: z.string() })),
    rows: z.array(z.record(z.string(), z.any())),
  }),
  component: ({ props }) => <CockpitDataTable {...props} />,
});

const TimelineView = defineComponent({
  name: "Timeline",
  description: "Chronological event timeline",
  props: z.object({
    events: z.array(z.object({
      time: z.string(),
      title: z.string(),
      description: z.string().optional(),
      status: z.enum(["success", "warning", "error", "info"]).optional(),
    })),
  }),
  component: ({ props }) => <CockpitTimeline {...props} />,
});

// ... more widgets

export const cockpitLibrary = createLibrary({
  root: "Dashboard",
  components: [DataTable, TimelineView, Chart, PeopleGrid, ...],
});
```

---

## Component Architecture

```
src/
  app/
    page.tsx                           # → Cockpit (homepage)
    chat/
      page.tsx                         # Existing chat
      [threadId]/page.tsx              # Existing chat thread

  components/
    cockpit/
      CockpitPage.tsx                  # Main cockpit layout
      DimensionBar.tsx                 # Horizontal strip of 6 pills
      DimensionPill.tsx                # Individual pill component
      QuerySynthesisBar.tsx            # Live query preview + free text input
      ResponseSurface.tsx              # OpenUI streaming renderer
      SuggestionRail.tsx               # Follow-up query suggestions

      dimensions/
        WhoDimension.tsx               # People picker popover
        WhatDimension.tsx              # Action/topic selector popover
        WhenDimension.tsx              # Timeline/date controls popover
        WhereDimension.tsx             # Source picker popover
        WhyDimension.tsx               # Intent/goal selector popover
        HowDimension.tsx               # Output format selector popover
        DimensionPopover.tsx           # Shared popover wrapper

      widgets/
        CockpitDataTable.tsx           # shadcn Table + TanStack Table
        CockpitTimeline.tsx            # Custom timeline (shadcn + Tailwind)
        CockpitChart.tsx               # Recharts wrapper
        CockpitPeopleGrid.tsx          # People/entity cards
        CockpitSummaryCard.tsx         # Summary/stats card
        CockpitSteps.tsx               # Stepper/reasoning display
        CockpitTree.tsx                # Hierarchical tree view
        CockpitComparison.tsx          # Side-by-side comparison

      openui/
        cockpit-library.ts             # OpenUI component library definition
        cockpit-renderer.tsx           # OpenUI Renderer wired to cockpit

    chat/
      ... (existing chat components)
```

---

## Interaction Flows

### Flow 1: Pill-driven query
```
User taps [WHO] → picks "Sarah Chen"
User taps [WHAT] → picks "pull requests"
User taps [WHEN] → picks "this week"

Query bar shows: "Show me Sarah Chen's pull requests from this week"
AI suggests: "...in GitHub" (auto-fills WHERE)

User hits [Execute ⚡]

Response surface streams in:
  → Data table of PRs (title, status, reviewers, comments)
  → Timeline of PR activity
  → Summary card with stats
```

### Flow 2: Free text → auto-populate dimensions
```
User types in query bar: "what did the backend team ship last sprint"

Dimensions auto-populate:
  WHO: "Backend Team" (lights up)
  WHAT: "shipped/merged" (lights up)
  WHEN: "Last sprint" (lights up)

User can adjust any dimension before executing.
```

### Flow 3: Cockpit → Chat escalation
```
User runs a cockpit query → sees response surface
Wants to ask follow-up questions conversationally

Clicks [Open in Chat →] or types a follow-up
→ Opens a chat thread pre-loaded with the cockpit context
→ AI Elements chat experience takes over
→ Back button returns to cockpit with state preserved
```

### Flow 4: Empty state / cold start
```
User lands on cockpit with no selections.

Shows:
  → "Good morning, JD" greeting
  → AI-suggested queries based on recent activity:
    ["Review yesterday's PRs"] ["Team standup prep"] ["Check production alerts"]
  → Recent cockpit queries (history)
  → Trending/active items across connected sources
```

---

## Build Phases

### Phase 1: Foundation (Week 1)
**Goal**: Cockpit page exists, dimension pills render, basic interaction works.

- [ ] Create `/cockpit` route and make it the homepage
- [ ] Install OpenUI (`@openuidev/react-lang`)
- [ ] Build `CockpitPage` layout (dimension bar + query bar + response area)
- [ ] Build `DimensionBar` with 6 `DimensionPill` components
- [ ] Build `DimensionPill` with empty/active/filled states
- [ ] Build `QuerySynthesisBar` with live text assembly
- [ ] Wire dimension state management (Zustand or React context)
- [ ] Empty state / cold start design

### Phase 2: Dimension Intelligence (Week 2)
**Goal**: Each dimension popover works with AI-powered suggestions.

- [ ] Build all 6 dimension popovers with appropriate controls
- [ ] Wire dimensions to Convex-backed AI autocomplete
- [ ] Implement cross-dimension filtering
- [ ] Implement free text → auto-populate dimensions (reverse flow)
- [ ] Add AI-suggested complete queries from partial input
- [ ] Keyboard shortcuts (Cmd+1 through Cmd+6 for dimensions)

### Phase 3: Response Surface + OpenUI (Week 3)
**Goal**: LLM responds with rich generative UI.

- [ ] Build cockpit widgets: DataTable, Timeline, Chart, PeopleGrid, SummaryCard, Steps, Tree, Comparison
- [ ] Define `cockpit-library.ts` registering all widgets with Zod schemas
- [ ] Wire OpenUI streaming `<Renderer>` in ResponseSurface
- [ ] Connect to Convex Agent threads for AI execution
- [ ] Stream responses via AI SDK with OpenUI Lang output
- [ ] Suggestion rail with follow-up queries

### Phase 4: Polish & Integration (Week 4)
**Goal**: Cockpit feels like a supercomputer. Chat integration works.

- [ ] Cockpit → Chat escalation flow
- [ ] Query history (recent cockpit queries)
- [ ] Animations & transitions (Motion — pill expand/collapse, response stream-in)
- [ ] Dark mode / visual polish — the "from the future" feel
- [ ] Mobile responsive layout
- [ ] Persistence (dimension selections survive refresh via Convex)
- [ ] Performance optimization

---

## Design Principles

1. **Cockpit, not chatbot** — Every pixel should feel like an instrument panel
2. **Dimensions are the primitives** — Who/What/When/Where/Why/How are the axes of every query
3. **Progressive disclosure** — Pills are minimal until tapped
4. **AI fills the gaps** — User provides partial intent, AI completes the rest
5. **Responses are instruments** — Data grids, timelines, charts — not paragraphs
6. **Chat is the escape hatch** — When structured control isn't enough, drop into freeform
7. **Speed** — Everything streams. No loading spinners, only progressive reveal.

---

## Open Questions

1. **Dimension persistence** — Do selections persist across sessions? Per-user Convex state?
2. **Multi-query** — Can the cockpit hold multiple response surfaces (tabbed? stacked?)?
3. **Connectors** — What sources does WHERE support at launch? (GitHub, Slack, Convex data?)
4. **Mobile** — Horizontal pill scroll? Bottom sheet dimensions? Desktop-first?
5. **Sharing** — Can you share a cockpit query (URL with dimension state encoded)?
6. **Agents** — Does each cockpit query spawn a Convex Agent thread? Or reuse one?

---

*Master plan v2 — 2026-04-18. ReUI dropped. OpenUI is the only new dependency.*
