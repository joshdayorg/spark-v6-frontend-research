# Cockpit Build Plan — Step by Step

> Grounded in the actual `spark-v6-frontend-jd` codebase as of 2026-04-18.

---

## Codebase Context

### Current State
- **Homepage** (`/`) is currently the chat new-thread page (`(app)/(chat)/page.tsx`)
- **Chat system is mock-based** — `ChatThreadsProvider` uses localStorage, simulated tool lifecycles, mock data. No real LLM or Convex wired yet.
- **Widget system exists** — `src/components/widgets/widget-renderer.tsx` with typed widgets: `stat-card`, `finding-card`, `bar-chart`, `line-chart`, `entity-profile`. Types in `src/lib/widgets/types.ts`.
- **35+ AI Elements** already installed in `src/components/ai-elements/`
- **Sidebar** has nav items: Chat, Pulse, Knowledge, Agents, Findings, Settings
- **Route structure**: `src/app/(app)/` with route groups. Chat lives in `(app)/(chat)/`.
- **App layout**: `(app)/layout.tsx` provides `SidebarProvider` + `AppSidebar` shell.
- **No Convex** in frontend repo — backend is in separate `spark-v6-codex` repo.

### Existing shadcn/ui Components
```
accordion, alert, alert-dialog, avatar, badge, button, button-group,
card, carousel, chart, collapsible, command, context-menu, dialog,
dropdown-menu, hover-card, input, input-group, popover, progress,
scroll-area, select, separator, sheet, sidebar, skeleton, spinner,
switch, table, tabs, textarea, toggle, tooltip
```

### Existing Hooks
- `use-chat-shortcuts.ts` — keyboard shortcuts for chat
- `use-messages.ts` — message helpers
- `use-mobile.ts` — responsive detection
- `use-scroll-to-bottom.ts`
- `use-smooth-text.ts`

### Existing Dependencies
- `ai` + `@ai-sdk/react` (Vercel AI SDK)
- `recharts` (charts)
- `@tanstack/react-table` (data tables)
- `motion` (animations)
- `cmdk` (command palette)
- `embla-carousel-react`
- `shiki` (syntax highlighting)
- `streamdown` (markdown streaming)

---

## Phase 1: Foundation (Week 1)

**Goal**: Cockpit page exists, dimension pills render, query bar works, you can see it.

### Step 1.1: Install OpenUI
```bash
cd ~/projects/spark-v6-frontend-jd
npm install @openuidev/react-lang
```
Verify it imports cleanly. No config needed — it's a React library.

### Step 1.2: Create Cockpit route group
Create a new route group parallel to `(chat)`:
```
src/app/(app)/
  (chat)/           # existing
    page.tsx        # current homepage (keep as chat)
    [threadId]/page.tsx
    layout.tsx
  (cockpit)/        # NEW
    page.tsx        # new cockpit homepage
    layout.tsx      # cockpit-specific layout (full width, no chat scroll)
```

**`(cockpit)/layout.tsx`**:
```tsx
import type { ReactNode } from "react";

export default function CockpitLayout({ children }: { children: ReactNode }) {
  return (
    <div className="flex h-svh flex-col overflow-hidden">
      {children}
    </div>
  );
}
```

### Step 1.3: Update routing — make Cockpit the homepage
Move the current chat page to `/chat` and make cockpit the root:

- Move `(chat)/page.tsx` logic → stays at `(chat)/page.tsx` (it's the "new chat" page)
- Create `(cockpit)/page.tsx` as the new homepage
- Update sidebar nav to change `{ href: "/", icon: MessageSquare, label: "Chat" }` → split into:
  - `{ href: "/", icon: Gauge, label: "Cockpit" }` (new homepage)
  - `{ href: "/chat", icon: MessageSquare, label: "Chat" }` (existing chat)
- Update `isChatPath()` in sidebar to handle the new routing

### Step 1.4: Build CockpitPage skeleton
**`(cockpit)/page.tsx`**:
```tsx
"use client";

import { DimensionBar } from "@/components/cockpit/DimensionBar";
import { QuerySynthesisBar } from "@/components/cockpit/QuerySynthesisBar";
import { ResponseSurface } from "@/components/cockpit/ResponseSurface";
import { SuggestionRail } from "@/components/cockpit/SuggestionRail";

export default function CockpitPage() {
  return (
    <div className="flex flex-1 flex-col">
      <DimensionBar />
      <QuerySynthesisBar />
      <ResponseSurface />
      <SuggestionRail />
    </div>
  );
}
```

### Step 1.5: Build DimensionBar + DimensionPill
Create `src/components/cockpit/`:

**`DimensionBar.tsx`** — Horizontal strip of 6 pills:
- Uses existing `badge.tsx` styling as base
- Each pill = `DimensionPill` component
- Flexbox row, centered, with gap
- Responsive: horizontal scroll on mobile

**`DimensionPill.tsx`** — Individual capsule:
- Props: `dimension`, `icon`, `label`, `value`, `onToggle`, `isActive`, `isFilled`
- States: empty (outline), active (glow + expanded), filled (compact value shown)
- Uses existing `popover.tsx` for expansion
- Pill click opens popover below
- For Phase 1: popovers show placeholder text ("WHO controls coming in Phase 2")

Dimension definitions:
```tsx
const DIMENSIONS = [
  { id: "who", label: "Who", icon: User, emoji: "👤" },
  { id: "what", label: "What", icon: Target, emoji: "🎯" },
  { id: "when", label: "When", icon: Clock, emoji: "🕐" },
  { id: "where", label: "Where", icon: MapPin, emoji: "📍" },
  { id: "why", label: "Why", icon: Lightbulb, emoji: "💡" },
  { id: "how", label: "How", icon: Settings, emoji: "⚙️" },
] as const;
```

### Step 1.6: Build QuerySynthesisBar
**`QuerySynthesisBar.tsx`**:
- Uses existing `input.tsx` or `textarea.tsx` as base
- Shows live-assembled query text from dimension selections
- Also accepts free text typing (dual-mode)
- Execute button on the right
- For Phase 1: static text preview, no AI synthesis yet

### Step 1.7: Build ResponseSurface (placeholder)
**`ResponseSurface.tsx`**:
- Empty state: centered message "Select dimensions and execute a query"
- Will hold the OpenUI `<Renderer>` in Phase 3
- For Phase 1: shows a placeholder card

### Step 1.8: Build SuggestionRail
**`SuggestionRail.tsx`**:
- Reuse pattern from existing `suggested-actions.tsx`
- Horizontal row of pill-shaped suggestions at the bottom
- For Phase 1: hardcoded starter suggestions

### Step 1.9: Wire dimension state
Create `src/hooks/use-cockpit-state.ts`:
- Zustand store (or React context) for dimension selections
- Shape: `{ who: string | null, what: string | null, when: string | null, where: string | null, why: string | null, how: string | null }`
- Actions: `setDimension(dim, value)`, `clearDimension(dim)`, `clearAll()`
- Derived: `synthesizedQuery` — assembles natural language from selections

### Step 1.10: Empty state / cold start
Adapt existing `Greeting` component pattern:
- "Good morning" / time-aware greeting
- Starter suggestion pills below
- Recent cockpit queries (empty for now)
- Use `motion` for entrance animations (same pattern as existing `Greeting`)

### Phase 1 Deliverable
✅ Cockpit is the homepage at `/`
✅ Chat moved to `/chat`
✅ 6 dimension pills render with empty/active/filled states
✅ Query synthesis bar shows assembled text
✅ Response surface placeholder
✅ Suggestion rail with starter queries
✅ Sidebar updated with Cockpit + Chat nav items
✅ OpenUI installed (not wired yet)

---

## Phase 2: Dimension Intelligence (Week 2)

**Goal**: Each dimension popover has real controls with AI-powered suggestions.

### Step 2.1: Build DimensionPopover shared wrapper
**`src/components/cockpit/dimensions/DimensionPopover.tsx`**:
- Wraps shadcn `Popover` with consistent styling
- Shared header (dimension label + close button)
- Search/filter input at top of each popover
- Content area for dimension-specific controls
- Footer with "Clear" and "Apply" buttons

### Step 2.2: Build WhoDimension
**`src/components/cockpit/dimensions/WhoDimension.tsx`**:
- People/entity picker
- Uses `command.tsx` (cmdk) for searchable list
- Items: people, teams, roles, entities
- Avatar + name + role display
- Multi-select with badges showing selected
- Data source: mock data initially, Convex query later

### Step 2.3: Build WhatDimension
**`src/components/cockpit/dimensions/WhatDimension.tsx`**:
- Action/topic selector
- Categorized list: Actions ("summarize", "compare", "find"), Topics ("pull requests", "deployments", "errors")
- Uses `command.tsx` with groups
- Single or multi-select

### Step 2.4: Build WhenDimension
**`src/components/cockpit/dimensions/WhenDimension.tsx`**:
- Presets: "Today", "Yesterday", "This week", "Last 7 days", "This month", "Last 30 days"
- Custom date range picker (uses existing `calendar` via shadcn — would need to add `calendar.tsx` from shadcn registry)
- Relative time selector
- Uses `button-group.tsx` for presets

### Step 2.5: Build WhereDimension
**`src/components/cockpit/dimensions/WhereDimension.tsx`**:
- Source/system picker
- List of connected sources: GitHub, Slack, Convex, Production Logs, etc.
- Icon + label for each source
- Multi-select with badges
- Uses `command.tsx`

### Step 2.6: Build WhyDimension
**`src/components/cockpit/dimensions/WhyDimension.tsx`**:
- Goal/intent picker
- Presets: "Debug an issue", "Prepare standup", "Audit security", "Research", "Plan sprint"
- Free text option for custom intent
- Single select

### Step 2.7: Build HowDimension
**`src/components/cockpit/dimensions/HowDimension.tsx`**:
- Output format selector
- Options: "Dashboard", "Summary", "Detailed report", "Comparison table", "Timeline view", "Raw data"
- Visual previews/icons for each format
- Uses `toggle-group` or radio-style selection

### Step 2.8: Cross-dimension filtering
Update `use-cockpit-state.ts`:
- When WHO changes, filter WHAT suggestions based on the entity type
- When WHEN changes, filter WHERE to show only sources with activity in that timeframe
- When WHY changes, auto-suggest HOW defaults ("Debug" → "Detailed report")
- Implement as derived state in the Zustand store

### Step 2.9: Free text → dimension auto-population
Add to `QuerySynthesisBar.tsx`:
- When user types free text, run a lightweight parse (can be regex-based for Phase 2, AI-based in Phase 3)
- Extract dimension hints: names → WHO, time words → WHEN, source names → WHERE
- Auto-fill dimension pills with extracted values
- Show "AI parsed" indicator on auto-filled pills

### Step 2.10: AI query suggestions
Add to `SuggestionRail.tsx`:
- Based on partially filled dimensions, suggest complete queries
- "You selected Sarah + last week → Show Sarah's PR activity this week?"
- For Phase 2: template-based suggestions (not AI-generated yet)

### Step 2.11: Keyboard shortcuts
Create `src/hooks/use-cockpit-shortcuts.ts`:
- `Cmd+1` through `Cmd+6` → toggle dimension popovers
- `Cmd+Enter` → execute query
- `Cmd+K` → focus query bar (reuse existing `command.tsx` pattern)
- `Escape` → close active popover

### Step 2.12: Add Calendar component (if missing)
```bash
npx shadcn@latest add calendar
```
Needed for the WHEN dimension date picker.

### Phase 2 Deliverable
✅ All 6 dimension popovers with real controls
✅ Searchable pickers (WHO, WHAT, WHERE use cmdk)
✅ Date presets + custom range (WHEN)
✅ Goal presets (WHY) and format selector (HOW)
✅ Cross-dimension filtering works
✅ Free text → auto-populates dimensions
✅ Template-based query suggestions
✅ Keyboard shortcuts

---

## Phase 3: Response Surface + OpenUI (Week 3)

**Goal**: LLM responds with rich generative UI composed from your widgets.

### Step 3.1: Build cockpit widgets
Create `src/components/cockpit/widgets/`:

Leverage and extend the **existing widget system** (`src/components/widgets/` + `src/lib/widgets/types.ts`):

| New Widget | Based on | What it renders |
|------------|----------|----------------|
| `CockpitDataTable.tsx` | existing `table.tsx` + `@tanstack/react-table` | Rich sortable/filterable data table |
| `CockpitTimeline.tsx` | NEW (shadcn + tailwind) | Vertical timeline with status dots, timestamps, descriptions |
| `CockpitChart.tsx` | existing `bar-chart.tsx` + `line-chart.tsx` | Unified chart wrapper (bar, line, area) |
| `CockpitPeopleGrid.tsx` | existing `entity-profile.tsx` | Grid of people cards with stats |
| `CockpitSummaryCard.tsx` | existing `stat-card.tsx` | KPI/metric summary |
| `CockpitSteps.tsx` | NEW (shadcn + tailwind) | Stepper/reasoning display |
| `CockpitTree.tsx` | NEW (shadcn + recursive) | Hierarchical tree (repos, files, channels) |
| `CockpitComparison.tsx` | NEW (shadcn + tailwind) | Side-by-side comparison layout |

### Step 3.2: Define widget Zod schemas
Create `src/components/cockpit/openui/widget-schemas.ts`:
- One Zod schema per widget matching its props
- Reuse existing types from `src/lib/widgets/types.ts` where possible
- Add new types for Timeline, Tree, Steps, Comparison

### Step 3.3: Create OpenUI component library
**`src/components/cockpit/openui/cockpit-library.ts`**:
```tsx
import { defineComponent, createLibrary } from "@openuidev/react-lang";
import { z } from "zod";
// import all widgets + schemas

const DataTable = defineComponent({
  name: "DataTable",
  description: "Sortable, filterable data table for structured results",
  props: dataTableSchema,
  component: ({ props }) => <CockpitDataTable {...props} />,
});

// ... define all 8 widgets

export const cockpitLibrary = createLibrary({
  root: "Dashboard",  // root component that wraps layout
  components: [DataTable, Timeline, Chart, PeopleGrid, SummaryCard, Steps, Tree, Comparison],
});
```

### Step 3.4: Generate system prompt
```tsx
// At build time or server-side:
const systemPrompt = cockpitLibrary.prompt({
  examples: [...], // few-shot examples of dimension queries → OpenUI Lang responses
  additionalRules: [
    "Always include a SummaryCard at the top of responses",
    "Use Timeline for WHEN-heavy queries",
    "Use PeopleGrid for WHO-heavy queries",
    "Use DataTable as the default for structured data",
  ],
});
```

### Step 3.5: Wire ResponseSurface to OpenUI Renderer
**Update `ResponseSurface.tsx`**:
```tsx
import { Renderer } from "@openuidev/react-lang";
import { cockpitLibrary } from "./openui/cockpit-library";

export function ResponseSurface({ response, isStreaming }) {
  if (!response) return <EmptyState />;
  
  return (
    <Renderer
      library={cockpitLibrary}
      response={response}
      isStreaming={isStreaming}
    />
  );
}
```

### Step 3.6: Create cockpit API route
**`src/app/api/cockpit/route.ts`** (or wire through existing chat API):
- Accepts structured query from cockpit (dimension selections + synthesized text)
- Attaches the OpenUI system prompt
- Calls LLM via Vercel AI SDK `streamText`
- Returns streaming response in OpenUI Lang format

**Note**: Since chat is currently mock-based, this will be the **first real LLM connection** in the frontend. Options:
- a) Wire directly to an LLM API via AI SDK (standalone, doesn't need Convex yet)
- b) Wire through Convex Agent thread (needs backend coordination)
- **Recommendation**: Start with (a) for speed. Swap to (b) when Convex integration happens.

### Step 3.7: Connect execute flow
Update `QuerySynthesisBar.tsx`:
- Execute button calls the cockpit API route
- Passes dimension selections + synthesized query
- Streams response into `ResponseSurface`
- Shows streaming state (shimmer/skeleton while loading)

### Step 3.8: Build Dashboard root component
The OpenUI library's `root` component — wraps and lays out child widgets:
```tsx
const Dashboard = defineComponent({
  name: "Dashboard",
  description: "Root layout that arranges response widgets",
  props: z.object({
    title: z.string().optional(),
    children: z.array(z.any()), // OpenUI handles child composition
  }),
  component: ({ props, children }) => (
    <div className="grid gap-4 p-4">
      {props.title && <h2 className="text-lg font-semibold">{props.title}</h2>}
      {children}
    </div>
  ),
});
```

### Step 3.9: Suggestion rail — AI-powered follow-ups
After a response renders, query the LLM for follow-up suggestions:
- "Dig deeper into Sarah's failed deployments"
- "Compare with last month"
- "Show this as a timeline instead"
- Display as pills in the suggestion rail

### Step 3.10: Response history (within session)
- Keep an in-memory stack of query → response pairs
- Allow user to scroll back through previous cockpit queries
- Each response is a separate `ResponseSurface` instance

### Phase 3 Deliverable
✅ 8 cockpit widgets built (DataTable, Timeline, Chart, PeopleGrid, SummaryCard, Steps, Tree, Comparison)
✅ OpenUI library registered with all widgets
✅ System prompt auto-generated from library
✅ Real LLM connected via API route + AI SDK `streamText`
✅ Execute button streams generative UI into response surface
✅ Widgets render progressively as tokens arrive
✅ AI-powered follow-up suggestions

---

## Phase 4: Polish & Integration (Week 4)

**Goal**: Cockpit feels like a supercomputer. Seamless chat integration. Production quality.

### Step 4.1: Cockpit → Chat escalation
- Add "Open in Chat →" button on response surface
- Creates a new chat thread pre-loaded with:
  - The cockpit query as the user message
  - The cockpit response context as system context
- Uses existing `useCreateThread` from `src/lib/chat-api`
- Router push to `/chat/[threadId]`
- Back button in chat returns to cockpit with state preserved

### Step 4.2: Animations & transitions
Use existing `motion` library (already installed):
- Pill expand/collapse: `motion.div` with `layout` prop
- Response surface stream-in: fade + slide-up per widget
- Query bar text assembly: character-by-character or word-by-word reveal
- Dimension auto-fill pulse: shimmer animation on pills
- Page transitions: smooth crossfade between cockpit and chat

### Step 4.3: Dark mode / visual polish
- Cockpit-specific dark theme adjustments
- Pill glow effects (subtle border glow when active)
- Response surface glass-morphism or subtle gradient background
- Ensure all widgets look polished in both light and dark
- "Supercomputer" visual language: monospace accents, subtle grid background, status indicators

### Step 4.4: Query history
- Store cockpit queries in localStorage (same pattern as `ChatThreadsProvider`)
- Show recent queries in empty state
- Allow re-running previous queries
- Display in sidebar under "Cockpit History" (new sidebar section)

### Step 4.5: Mobile responsive
- Dimension bar: horizontal scroll with snap points
- Dimension popovers: bottom sheet on mobile (use existing `sheet.tsx` instead of `popover.tsx`)
- Response surface: single column stack
- Query bar: full width, sticky at top
- Use existing `use-mobile.ts` hook for breakpoint detection

### Step 4.6: Persistence
- Dimension selections persist across page navigations
- Store in localStorage initially (match existing pattern)
- Later: Convex per-user state when backend is wired

### Step 4.7: Performance optimization
- Lazy load widgets (React.lazy + Suspense)
- Memoize dimension popovers
- Debounce cross-dimension filtering
- Streaming renderer already handles progressive rendering (OpenUI)

### Step 4.8: Error states
- LLM API failure: show retry button in response surface
- Partial response: show what rendered + error indicator
- Network offline: show cached last response
- Invalid dimension combinations: show helpful hint

### Step 4.9: Accessibility
- Dimension pills are keyboard navigable (arrow keys)
- Popovers trap focus correctly (Radix handles this)
- Screen reader labels for all interactive elements
- Execute button has proper aria-label
- Response surface announces new content

### Step 4.10: Update sidebar
- Add Cockpit nav item with cockpit icon (Gauge from lucide)
- Add "Cockpit History" section (below Thread History when on cockpit)
- "New Query" button (like "New Chat")
- Visual distinction between cockpit and chat sidebar states

### Phase 4 Deliverable
✅ Cockpit → Chat escalation works seamlessly
✅ Smooth animations throughout
✅ Dark mode polished
✅ Query history persisted
✅ Mobile responsive
✅ Error states handled
✅ Keyboard accessible
✅ Sidebar updated with cockpit navigation
✅ **Feels like a supercomputer**

---

## Files Changed / Created Summary

### New files (~30)
```
src/app/(app)/(cockpit)/
  layout.tsx
  page.tsx

src/components/cockpit/
  CockpitPage.tsx
  DimensionBar.tsx
  DimensionPill.tsx
  QuerySynthesisBar.tsx
  ResponseSurface.tsx
  SuggestionRail.tsx
  CockpitEmptyState.tsx

  dimensions/
    DimensionPopover.tsx
    WhoDimension.tsx
    WhatDimension.tsx
    WhenDimension.tsx
    WhereDimension.tsx
    WhyDimension.tsx
    HowDimension.tsx

  widgets/
    CockpitDataTable.tsx
    CockpitTimeline.tsx
    CockpitChart.tsx
    CockpitPeopleGrid.tsx
    CockpitSummaryCard.tsx
    CockpitSteps.tsx
    CockpitTree.tsx
    CockpitComparison.tsx

  openui/
    cockpit-library.ts
    cockpit-renderer.tsx
    widget-schemas.ts

src/hooks/
  use-cockpit-state.ts
  use-cockpit-shortcuts.ts

src/app/api/cockpit/
  route.ts
```

### Modified files (~5)
```
src/components/shell/sidebar.tsx        # Add Cockpit nav item
src/app/(app)/(chat)/page.tsx           # May adjust if routing changes
package.json                            # Add @openuidev/react-lang
src/lib/widgets/types.ts                # Extend with new widget types
```

---

## Dependencies Added

| Package | Phase | Why |
|---------|-------|-----|
| `@openuidev/react-lang` | Phase 1 (install), Phase 3 (wire) | Generative UI protocol |
| `zustand` | Phase 1 | Cockpit dimension state management (optional — could use React context) |
| `@shadcn/calendar` | Phase 2 | WHEN dimension date picker (if not already added) |

Total new deps: **1-3** (OpenUI is the only required one)

---

*Plan grounded in codebase review of spark-v6-frontend-jd, 2026-04-18.*
