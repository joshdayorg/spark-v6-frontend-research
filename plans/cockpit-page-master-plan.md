# Cockpit Page — Master Plan

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

## Page Anatomy

```
┌─────────────────────────────────────────────────────────────────┐
│  SPARK COCKPIT                                    [⚙] [Chat →] │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    DIMENSION BAR                         │    │
│  │                                                         │    │
│  │  ┌─────┐  ┌──────┐  ┌──────┐  ┌───────┐  ┌─────┐  ┌─────┐│
│  │  │ WHO │  │ WHAT │  │ WHEN │  │ WHERE │  │ WHY │  │ HOW ││
│  │  └─────┘  └──────┘  └──────┘  └───────┘  └─────┘  └─────┘│
│  │                                                         │    │
│  │  Small pills / capsules. Tap to expand. Glow when active│    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                  QUERY SYNTHESIS BAR                     │    │
│  │                                                         │    │
│  │  "Show me [Sarah's] [deployment history] from [last     │    │
│  │   week] in [GitHub] to [debug the outage]"              │    │
│  │                                                         │    │
│  │  Live-updating as you interact with dimension pills.    │    │
│  │  Also typeable — free text auto-populates dimensions.   │    │
│  │                                       [Execute ⚡]      │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                  RESPONSE SURFACE                        │    │
│  │                                                         │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │    │
│  │  │  Data Grid   │  │   Timeline   │  │    Cards     │  │    │
│  │  │  (ReUI)      │  │   (ReUI)     │  │   (shadcn)   │  │    │
│  │  └──────────────┘  └──────────────┘  └──────────────┘  │    │
│  │                                                         │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │    │
│  │  │   Kanban     │  │    Tree      │  │    Chart     │  │    │
│  │  │   (ReUI)     │  │   (ReUI)     │  │  (Recharts)  │  │    │
│  │  └──────────────┘  └──────────────┘  └──────────────┘  │    │
│  │                                                         │    │
│  │  Generative — LLM picks & composes via OpenUI.          │    │
│  │  Streams in real-time as response arrives.              │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                  SUGGESTION RAIL                         │    │
│  │  [Related query 1]  [Related query 2]  [Dig deeper →]   │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

---

## The Six Dimensions

Each dimension is a **pill** — a small capsule button in the dimension bar. Tapping it opens an expanded panel/popover with AI-powered controls.

| Dimension | Pill Label | Expanded Controls | Example Selections |
|-----------|-----------|-------------------|--------------------|
| **WHO** | 👤 Who | People picker, @mentions, entity search, role filter | "Sarah Chen", "Backend Team", "Customer: Acme" |
| **WHAT** | 🎯 What | Action/topic selector, verb palette, content type | "deployments", "pull requests", "errors", "summarize" |
| **WHEN** | 🕐 When | Timeline scrubber, date range picker, recency presets ("today", "this week", "last month") | "Last 7 days", "Since v2.3 release", "Yesterday" |
| **WHERE** | 📍 Where | Source picker, repo/channel/system selector, data source | "GitHub", "Slack #engineering", "Production logs" |
| **WHY** | 💡 Why | Goal/intent selector, reasoning depth, purpose | "Debug outage", "Prepare standup", "Audit security" |
| **HOW** | ⚙️ How | Output format, detail level, response style | "Dashboard view", "Brief summary", "Detailed report", "Comparison table" |

### Pill States

- **Empty** — subtle outline, muted icon, no selection
- **Active** — expanded popover is open, pill glows/highlights
- **Filled** — dimension has a selection, pill shows compact summary ("👤 Sarah", "🕐 Last 7d")
- **AI-suggested** — subtle pulse/shimmer when AI recommends filling this dimension

### Cross-dimension intelligence

- Selecting WHO → narrows WHAT suggestions ("Sarah" → shows her repos, her PRs, her recent work)
- Selecting WHEN → filters WHERE options ("Last 7 days" → shows only sources with recent activity)
- Selecting WHY → shapes HOW defaults ("Debug outage" → defaults to "Detailed report" format)
- AI proposes complete queries from partial input ("You selected Sarah + last week → Did you mean her PR review status?")

---

## Technology Stack

### Already Have (keep)
- **Next.js 16** App Router — page routing, server components
- **Vercel AI SDK** (`ai` + `@ai-sdk/react`) — `useChat`, `streamText`, model routing, tool execution
- **AI Elements** — conversation components (for when cockpit escalates to chat)
- **shadcn/ui** — base primitives (buttons, popovers, command palette, cards)
- **Tailwind CSS v4** — styling
- **Recharts** — charts
- **Convex** — real-time backend, agent threads, subscriptions
- **TanStack Table** — data tables

### Adding
- **[ReUI](https://github.com/keenthemes/reui)** — Dashboard-grade components built on shadcn:
  - Data Grid (advanced, beyond TanStack Table)
  - Kanban board
  - Timeline
  - Tree view
  - Filters (faceted, stackable)
  - Sortable lists
  - Stepper
  - Nested layouts
- **[OpenUI](https://github.com/thesysdev/openui)** — Generative UI protocol:
  - `@openuidev/react-lang` — define component library with Zod, streaming parser/renderer
  - `@openuidev/react-ui` — prebuilt layouts (optional, may not need if we use our own)
  - Auto-generates system prompt from component definitions
  - LLM responds in OpenUI Lang (67% fewer tokens than JSON)
  - Streaming renderer displays components as tokens arrive

---

## Component Architecture

```
src/
  app/
    page.tsx                           # → Cockpit (homepage)
    cockpit/
      page.tsx                         # Cockpit route (alias)
    chat/
      page.tsx                         # Existing chat
      [threadId]/
        page.tsx                       # Existing chat thread

  components/
    cockpit/
      CockpitPage.tsx                  # Main cockpit layout
      DimensionBar.tsx                 # Horizontal strip of 6 pills
      DimensionPill.tsx                # Individual pill (shared component)
      QuerySynthesisBar.tsx            # Live query preview + free text input
      ResponseSurface.tsx              # Generative response area (OpenUI renderer)
      SuggestionRail.tsx               # Follow-up query suggestions
      
      dimensions/
        WhoDimension.tsx               # People picker popover
        WhatDimension.tsx              # Action/topic selector popover
        WhenDimension.tsx              # Timeline/date controls popover
        WhereDimension.tsx             # Source picker popover
        WhyDimension.tsx               # Intent/goal selector popover
        HowDimension.tsx               # Output format selector popover
        DimensionPopover.tsx           # Shared popover wrapper
      
      response-widgets/
        PeopleGrid.tsx                 # WHO-heavy response → ReUI Data Grid
        ActionKanban.tsx               # WHAT-heavy response → ReUI Kanban
        EventTimeline.tsx              # WHEN-heavy response → ReUI Timeline
        SourceTree.tsx                 # WHERE-heavy response → ReUI Tree
        ReasoningSteps.tsx             # WHY-heavy response → ReUI Stepper
        DashboardComposite.tsx         # Multi-dimension → composed layout
        DataChart.tsx                  # Quantitative data → Recharts
        ComparisonTable.tsx            # HOW="compare" → side-by-side
      
      openui/
        cockpit-library.ts             # OpenUI component library definition
        cockpit-renderer.tsx           # OpenUI streaming renderer config

    chat/
      ... (existing chat components)
```

---

## OpenUI Integration

### Component Library Definition

```typescript
// cockpit-library.ts
import { z } from "zod";
import { defineComponent, createLibrary } from "@openuidev/react-lang";
import { PeopleGrid } from "../response-widgets/PeopleGrid";
import { EventTimeline } from "../response-widgets/EventTimeline";
import { ActionKanban } from "../response-widgets/ActionKanban";
// ... etc

const PeopleGridComponent = defineComponent({
  name: "PeopleGrid",
  description: "Display people/entities in a rich data grid",
  props: z.object({
    people: z.array(z.object({
      name: z.string(),
      role: z.string().optional(),
      avatar: z.string().url().optional(),
      stats: z.record(z.string(), z.number()).optional(),
    })),
    title: z.string().optional(),
  }),
  component: ({ props }) => <PeopleGrid {...props} />,
});

// ... define all response widgets

export const cockpitLibrary = createLibrary({
  root: "Dashboard",
  components: [PeopleGridComponent, TimelineComponent, KanbanComponent, ...],
});
```

### Flow

1. User manipulates dimension pills → selections form a structured intent
2. `QuerySynthesisBar` assembles natural language query from selections
3. Query sent to LLM with `cockpitLibrary.prompt()` as system prompt
4. LLM responds in OpenUI Lang, referencing registered components
5. `ResponseSurface` uses OpenUI's streaming `<Renderer>` to display live
6. User sees ReUI widgets (data grids, timelines, kanban) composing in real-time

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
  → Data Grid of PRs (title, status, reviewers, comments)
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

- [ ] Create `/cockpit` route (and make it the homepage)
- [ ] Install ReUI into the project
- [ ] Install OpenUI (`@openuidev/react-lang`) into the project
- [ ] Build `CockpitPage` layout (dimension bar + query bar + response area)
- [ ] Build `DimensionBar` with 6 `DimensionPill` components
- [ ] Build `DimensionPill` with empty/active/filled states
- [ ] Build `QuerySynthesisBar` with live text assembly
- [ ] Wire basic state management (Zustand store for dimension selections)
- [ ] Empty state / cold start design

### Phase 2: Dimension Intelligence (Week 2)
**Goal**: Each dimension popover works with AI-powered suggestions.

- [ ] Build all 6 dimension popovers with appropriate controls
- [ ] Wire dimensions to Convex-backed AI autocomplete
- [ ] Implement cross-dimension filtering (WHO selection narrows WHAT)
- [ ] Implement free text → auto-populate dimensions (reverse flow)
- [ ] Add AI-suggested complete queries from partial input
- [ ] Keyboard shortcuts (Cmd+1 through Cmd+6 for dimensions)

### Phase 3: Response Surface (Week 3)
**Goal**: LLM responds with rich generative UI via OpenUI.

- [ ] Define `cockpit-library.ts` with all response widget components
- [ ] Build response widgets: PeopleGrid, EventTimeline, ActionKanban, SourceTree, ReasoningSteps, DataChart, ComparisonTable
- [ ] Wire OpenUI streaming renderer in ResponseSurface
- [ ] Connect to Convex Agent threads for AI execution
- [ ] Stream responses via AI SDK with OpenUI Lang output
- [ ] Suggestion rail with follow-up queries

### Phase 4: Polish & Integration (Week 4)
**Goal**: Cockpit feels like a supercomputer. Chat integration works.

- [ ] Cockpit → Chat escalation flow
- [ ] Query history (recent cockpit queries)
- [ ] Animations & transitions (pill expand/collapse, response stream-in)
- [ ] Dark mode / visual polish — the "from the future" feel
- [ ] Mobile responsive layout (dimension pills as horizontal scroll or bottom sheet)
- [ ] Performance optimization (Convex subscriptions, streaming)
- [ ] Persistence (dimension selections survive page refresh via Convex)

---

## Design Principles

1. **Cockpit, not chatbot** — Every pixel should feel like an instrument panel, not a messaging app
2. **Dimensions are the primitives** — Who/What/When/Where/Why/How are the fundamental axes of every query
3. **Progressive disclosure** — Pills are minimal until tapped. Complexity reveals on demand.
4. **AI fills the gaps** — User provides partial intent, AI completes the rest
5. **Responses are instruments** — Data grids, timelines, kanban boards — not paragraphs
6. **Chat is the escape hatch** — When structured control isn't enough, drop into freeform conversation
7. **Speed** — Everything streams. Everything is real-time. No loading spinners, only progressive reveal.

---

## Open Questions

1. **Dimension persistence** — Do selections persist across sessions? Per-user Convex state?
2. **Multi-query** — Can the cockpit hold multiple response surfaces (tabbed? stacked?)?
3. **Connectors** — What sources does WHERE dimension support at launch? (GitHub, Slack, Convex data?)
4. **Mobile** — Horizontal pill scroll? Bottom sheet dimensions? Or cockpit is desktop-first?
5. **Sharing** — Can you share a cockpit query (URL with dimension state encoded)?
6. **Agents** — Does each cockpit query spawn a Convex Agent thread? Or reuse one?

---

*Master plan created 2026-04-18. Ready to execute on your go.*
