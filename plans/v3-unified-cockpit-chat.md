# v3: Chat IS the Cockpit

> The home screen is both the cockpit and the chat entry point. The 5W1H dimensions are live intelligence AND input controls. The business is a sentient being communicating its own state.

---

## Why This Is Better

The v2 plan had Cockpit and Chat as separate pages. That created:
- A routing refactor (moving chat to `/chat/*`)
- A cockpit → chat escalation flow
- Duplicate state management
- A confusing "where do I go?" decision for users

The v3 direction eliminates all of that. The home screen at `/` stays exactly where it is. We're upgrading it, not replacing it.

### What changes from v2:
| v2 (separate) | v3 (unified) |
|---|---|
| Cockpit at `/`, Chat at `/chat` | Home screen IS the cockpit. Chat threads still at `/[threadId]`. |
| Routing refactor needed | **No routing changes.** |
| Empty cockpit until user interacts | **5W1H pre-populated with live intelligence on load.** |
| Dimension pills are input-only controls | **Dimensions are both displays (intelligence) AND inputs (controls).** |
| Cockpit → Chat escalation flow | **Interacting with dimensions starts a chat thread naturally.** |
| Separate cockpit state management | **Reuses existing chat thread creation flow.** |

---

## The Home Screen

The current home screen (`(chat)/page.tsx`) shows:
- Greeting: "What would you like to know?"
- Chat input (MultimodalInput)
- Suggested actions

The new home screen shows the **business communicating through 5W1H**:

```
┌───────────────────────────────────────────────────────────────┐
│                                                               │
│  Good morning, JD.                                            │
│  Here's what your business is telling you:                    │
│                                                               │
│  ┌─────────────────────────────┐┌────────────────────────────┐│
│  │ 👤 WHO                        ││ 🎯 WHAT                      ││
│  │                             ││                             ││
│  │ Sarah deployed 3x today    ││ 2 PRs awaiting review      ││
│  │ Marcus hasn't committed    ││ Staging build failing      ││
│  │ 🟢 4 agents active           ││ Customer ticket spike      ││
│  │                             ││                             ││
│  │ 🔍 Search people...          ││ 🔍 Search actions...         ││
│  └─────────────────────────────┘└────────────────────────────┘│
│                                                               │
│  ┌─────────────────────────────┐┌────────────────────────────┐│
│  │ 🕐 WHEN                       ││ 📍 WHERE                     ││
│  │                             ││                             ││
│  │ Sprint ends in 3 days      ││ GitHub: 12 open PRs        ││
│  │ Standup in 45 min          ││ Slack: 3 unread threads    ││
│  │ Deploy window 2pm-4pm      ││ Prod: all systems green    ││
│  │                             ││                             ││
│  │ 🔍 Search timeline...        ││ 🔍 Search sources...         ││
│  └─────────────────────────────┘└────────────────────────────┘│
│                                                               │
│  ┌─────────────────────────────┐┌────────────────────────────┐│
│  │ 💡 WHY                        ││ ⚙️ HOW                       ││
│  │                             ││                             ││
│  │ Q2 goal: 98% uptime        ││ CI/CD: 14 min avg build   ││
│  │ Currently: 99.2% ✅        ││ Test coverage: 73% ⚠️     ││
│  │ Churn risk: Acme Corp      ││ Deploy frequency: 4x/day  ││
│  │                             ││                             ││
│  │ 🔍 Search goals...           ││ 🔍 Search processes...       ││
│  └─────────────────────────────┘└────────────────────────────┘│
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  Or just ask anything:                                     ││
│  │  [____________________________________________] [⚡]       ││
│  └─────────────────────────────────────────────────────────┘│
└───────────────────────────────────────────────────────────────┘
```

---

## The Business as a Sentient Being

The 5W1H cards aren't empty controls waiting for input. They're the business **talking to you**.

### What each dimension communicates:

| Dimension | The business says... | Tone |
|-----------|---------------------|------|
| **WHO** | "Here's your team. Sarah's been crushing it. Marcus might need a check-in. Your agents are running." | Awareness of people |
| **WHAT** | "Here's what's happening right now. These PRs need you. This build is broken. Customers are reaching out more than usual." | Situation report |
| **WHEN** | "Here's your timeline. Sprint ends soon. You have a deploy window coming up. Standup is in 45 minutes." | Time awareness |
| **WHERE** | "Here's the state of your systems. GitHub is busy. Slack has unread threads. Production is healthy." | System health |
| **WHY** | "Here's what you're working toward. Your uptime goal is on track. But watch Acme Corp — churn risk." | Purpose + risk |
| **HOW** | "Here's how your machine is running. Builds are fast. Coverage could be better. You're deploying frequently." | Process health |

Each card has two modes:
1. **Vitals mode** (default) — The business showing its current state. Auto-populated. Updated in real-time.
2. **Search mode** (on click) — The user drilling in. Click the card or the search bar within it → Google-style autocomplete with suggestions contextualized to that dimension.

### The visual language of a sentient business:

- **Status indicators**: Green dots (🟢), yellow warnings (⚠️), red alerts (🟥) — the business showing where it's healthy and where it hurts
- **Natural language signals**: Not "3 PRs open" but "2 PRs need your review" — the business is asking for help, not reporting metrics
- **Urgency hierarchy**: The most important signals float to the top of each card. The business knows what's most pressing.
- **Breathing/pulsing**: Cards with items that need attention have a subtle pulse (like a heartbeat). Cards that are healthy are calm.
- **The greeting adapts**: "Good morning JD" + context. Monday morning: "Here's what happened over the weekend." End of sprint: "3 days left — here's where things stand."

---

## The 5W1H Interaction Pattern

### State 1: Vitals (default — on page load)

Each 5W1H card shows 2-4 intelligence signals powered by OpenUI:
- Auto-populated based on user identity, org context, connected sources
- Generated server-side on page load (or via Convex real-time subscription)
- Each signal is a mini OpenUI-rendered component (stat card, status line, chart sparkline)
- Updated in real-time as data changes

### State 2: Search (on card click or search bar focus)

```
User clicks the WHO card or its search bar:

┌───────────────────────────────────────────────┐
│ 👤 WHO                                             │
│                                                   │
│ ┌───────────────────────────────────────────┐ │
│ │ Sa|                                           │ │
│ └───────────────────────────────────────────┘ │
│                                                   │
│  👤 Sarah Chen — deployed 3x today                 │
│  👤 Sam Ortiz — reviewing PR #482                  │
│  👤 Sandra Lee — offline since yesterday            │
│                                                   │
│  Suggested:                                       │
│  [Sarah's deploy history] [Backend team status]   │
└───────────────────────────────────────────────┘
```

The card **expands** in place. The vitals fade out, the search input appears, autocomplete results stream in. Results are contextual — not just names, but names + what they're doing right now.

### State 3: Query built → Thread created

When the user selects from a dimension or clicks a suggestion:
- A chat thread is created (using existing `useCreateThread`)
- The structured query becomes the first user message
- Router pushes to `/[threadId]`
- The AI responds with rich OpenUI content in the thread
- The home screen is one back-button away

Or: user can click any of the intelligence signals directly (e.g., click "2 PRs awaiting review") and that becomes the thread prompt.

### State 4: Free text (the existing chat input)

The chat input stays at the bottom. User can always just type freely. The 5W1H grid is above it. Free text and structured dimensions coexist.

---

## OpenUI Integration (Revised)

### Two uses of OpenUI on the home screen:

**Use 1: Vitals generation (server-side, on page load)**
- Server-side API route generates 5W1H vitals for the current user
- Each dimension's vitals are small OpenUI-rendered components (stat lines, status dots, sparklines)
- Uses a `vitalsLibrary` with lightweight components
- Refreshes periodically or via Convex subscription

**Use 2: Response rendering (in chat threads)**
- When a cockpit query starts a chat thread, the response uses a `responseLibrary` with richer widgets
- Same as the v2 plan: data tables, timelines, charts, etc.
- The `<Renderer>` lives inside the chat message display

### Two OpenUI libraries:

```typescript
// Lightweight components for the 5W1H vitals cards
export const vitalsLibrary = createLibrary({
  root: "VitalsCard",
  components: [StatusLine, MetricBadge, Sparkline, AlertSignal, PersonStatus],
});

// Rich components for chat thread responses  
export const responseLibrary = createLibrary({
  root: "Dashboard",
  components: [DataTable, Timeline, Chart, PeopleGrid, SummaryCard, Steps, Tree, Comparison],
});
```

---

## Component Architecture (Revised)

```
src/
  app/(app)/(chat)/
    page.tsx                          # UPGRADED — home screen with 5W1H + chat input
    [threadId]/page.tsx               # Existing thread view (no changes)
    layout.tsx                        # Existing (no changes)

  components/
    cockpit/                          # NEW — 5W1H home screen components
      HomeScreen.tsx                  # Orchestrates greeting + 5W1H grid + chat input
      DimensionGrid.tsx               # 2x3 grid of dimension cards
      DimensionCard.tsx               # Individual 5W1H card (vitals + search modes)
      DimensionSearch.tsx             # Autocomplete/search input within a card
      VitalsRenderer.tsx              # OpenUI Renderer for vitals content
      CockpitGreeting.tsx             # Contextual greeting (replaces current Greeting)

      vitals/                         # OpenUI vitals components
        StatusLine.tsx                # "Sarah deployed 3x today"
        MetricBadge.tsx               # "99.2% ✅"
        Sparkline.tsx                 # Tiny inline chart
        AlertSignal.tsx               # "⚠️ Test coverage: 73%"
        PersonStatus.tsx              # Avatar + name + activity
        vitals-library.ts             # OpenUI library for vitals

      widgets/                        # OpenUI response widgets (for chat threads)
        CockpitDataTable.tsx
        CockpitTimeline.tsx
        CockpitChart.tsx
        CockpitPeopleGrid.tsx
        CockpitSummaryCard.tsx
        CockpitSteps.tsx
        CockpitTree.tsx
        CockpitComparison.tsx
        response-library.ts           # OpenUI library for responses

    chat/
      greeting.tsx                    # REPLACED by CockpitGreeting
      suggested-actions.tsx           # REPLACED by DimensionGrid signals
      ... (rest unchanged)
```

### What changes in existing files:

| File | Change |
|------|--------|
| `(chat)/page.tsx` | Replace `Greeting` + `SuggestedActions` with `HomeScreen` (which includes `DimensionGrid` + the existing `MultimodalInput`) |
| `shell/sidebar.tsx` | No routing changes needed. Maybe add a "Cockpit" label or icon to the home nav item. |
| `package.json` | Add `@openuidev/react-lang` |

That's it. **3 files modified.** Everything else is new.

---

## Revised Build Phases

### Phase 1: The Living Home Screen (Week 1)

**Goal**: Home screen shows 5W1H cards with mock intelligence. Cards are clickable.

| Step | Task | Details |
|------|------|---------|
| 1.1 | Install OpenUI | `npm install @openuidev/react-lang` |
| 1.2 | Create `HomeScreen.tsx` | Replaces current greeting + suggestions. Includes `DimensionGrid` + `MultimodalInput`. |
| 1.3 | Create `DimensionGrid.tsx` | 2x3 responsive grid. Each cell is a `DimensionCard`. |
| 1.4 | Create `DimensionCard.tsx` | Two modes: vitals (default) and search (on click). Vitals show 2-4 mock signals. Search shows placeholder input. Uses `card.tsx` as base. |
| 1.5 | Create `CockpitGreeting.tsx` | Contextual greeting that replaces the generic "What would you like to know?" Time-aware, context-aware. |
| 1.6 | Create mock vitals data | `src/lib/mock-data/vitals.ts` — hardcoded signals for each dimension. Enough to look real. |
| 1.7 | Wire into `(chat)/page.tsx` | Replace `Greeting` + `SuggestedActions` imports with `HomeScreen`. Keep `MultimodalInput` at bottom. |
| 1.8 | Card click → search mode | Clicking a card expands it, shows search input, fades out vitals. Uses `motion` for animation. |
| 1.9 | Clicking a signal → creates thread | Clicking "2 PRs awaiting review" calls `createThread("Show me the PRs awaiting review")` and navigates to the thread. Uses existing `useCreateThread`. |
| 1.10 | Status indicators | Green/yellow/red dots, pulse animation on cards needing attention. Tailwind + motion. |

**Phase 1 Deliverable**: Home screen feels alive. 5W1H cards show mock intelligence. Clicking signals starts chat threads. Chat input still works at the bottom. No routing changes.

### Phase 2: Dimension Search (Week 2)

**Goal**: Each dimension card's search mode works with autocomplete suggestions.

| Step | Task | Details |
|------|------|---------|
| 2.1 | Create `DimensionSearch.tsx` | Shared autocomplete component. Uses `command.tsx` (cmdk) inside the expanded card. |
| 2.2 | WHO search | Search people/teams/entities. Avatar + name + current activity. Results from mock data. |
| 2.3 | WHAT search | Search actions/topics. Categorized: Actions, Topics, Recent. |
| 2.4 | WHEN search | Time presets + calendar date picker. Install `npx shadcn@latest add calendar`. |
| 2.5 | WHERE search | Search sources/systems. Icon + name + health status. |
| 2.6 | WHY search | Search goals/intents. Presets + free text. |
| 2.7 | HOW search | Output format selector. Visual thumbnails for each format. |
| 2.8 | Selection → thread creation | Selecting from any dimension search builds a structured query and creates a thread. Multiple dimensions can be selected before executing. |
| 2.9 | Cross-dimension context | Selecting WHO shows their relevant WHAT items. Selecting WHEN filters to that timeframe. Implemented as filtered mock data. |
| 2.10 | Mobile: card → sheet | On mobile, clicking a dimension card opens a `sheet.tsx` (bottom drawer) instead of expanding in-grid. Uses `use-mobile.ts`. |
| 2.11 | Keyboard shortcuts | `Cmd+1` through `Cmd+6` focus dimension cards. `Cmd+K` focuses the chat input. |
| 2.12 | Multi-dimension query builder | Small pills above the chat input showing selected dimensions. "[WHO: Sarah] [WHAT: PRs] [WHEN: This week] → [Ask ⚡]" |

**Phase 2 Deliverable**: Every dimension card has working search/autocomplete. Selecting from dimensions builds queries and starts threads. Mobile works.

### Phase 3: Live Intelligence + OpenUI (Week 3)

**Goal**: Vitals are AI-generated via OpenUI. Chat responses use rich widgets.

| Step | Task | Details |
|------|------|---------|
| 3.1 | Configure LLM provider | Install `@ai-sdk/openai` (or provider of choice). Add API key to `.env.local`. Verify `streamText` works standalone. |
| 3.2 | Build vitals OpenUI components | `StatusLine`, `MetricBadge`, `Sparkline`, `AlertSignal`, `PersonStatus` — lightweight, card-sized. |
| 3.3 | Create `vitals-library.ts` | Register vitals components with Zod schemas. Generate system prompt. |
| 3.4 | Create vitals API route | `src/app/api/vitals/route.ts` — accepts user context, returns OpenUI Lang for each dimension's vitals. |
| 3.5 | Wire `VitalsRenderer.tsx` | Each `DimensionCard` uses `<Renderer library={vitalsLibrary} response={vitalsResponse} />` for its content. Replaces mock data from Phase 1. |
| 3.6 | Build response widgets | `CockpitDataTable`, `CockpitTimeline`, `CockpitChart`, `CockpitPeopleGrid`, `CockpitSummaryCard`, `CockpitSteps`, `CockpitTree`, `CockpitComparison`. Standalone components (NOT dependent on Streamdown/MessagePartsContext). |
| 3.7 | Create `response-library.ts` | Register response widgets. Generate system prompt with examples + rules. |
| 3.8 | Wire response rendering in chat | When a cockpit-originated thread gets an AI response, render it via OpenUI `<Renderer>` instead of (or alongside) Streamdown markdown. |
| 3.9 | Vitals refresh | Vitals update periodically (polling) or via Convex subscription when available. Each card shows a subtle refresh indicator. |
| 3.10 | Dimension search → AI-powered | Replace mock autocomplete with AI-generated suggestions based on real data. Uses a lightweight LLM call per dimension. |

**Phase 3 Deliverable**: Home screen vitals are live AI-generated intelligence. Chat responses render rich widgets. The business is actually communicating.

### Phase 4: The Supercomputer Feel (Week 4)

| Step | Task | Details |
|------|------|---------|
| 4.1 | Greeting intelligence | Greeting adapts: Monday morning recap, end-of-sprint urgency, post-incident debrief. Generated by LLM based on context. |
| 4.2 | Vitals priority sorting | Most urgent/important signals float to top of each card. AI determines priority. Cards with alerts pulse subtly. |
| 4.3 | Card animations | Vitals content animates in on page load (staggered, per-card). Search mode transitions are fluid. Use `motion` `layoutId` for shared element transitions. |
| 4.4 | Dark mode polish | Cockpit-specific dark theme. Subtle glow on status indicators. Glass-morphism on cards. Grid background pattern. |
| 4.5 | Query history | Recent cockpit queries shown below the 5W1H grid (or in sidebar). Click to re-run. |
| 4.6 | Card detail expansion | A dimension card can expand to full-width for deeper exploration without leaving the home screen. |
| 4.7 | Real-time signals | Vitals update live. New signals slide in with animation. Resolved alerts fade out. The business breathes. |
| 4.8 | Sound design (optional) | Subtle audio cues for alerts, completions. Very optional but adds to the "sentient" feel. |
| 4.9 | Persistence | Dimension search preferences persist. Last-viewed vitals cached. Fast page loads. |
| 4.10 | Performance | Lazy load dimension search components. Cache vitals responses. Debounce autocomplete. Skeleton states for everything. |

**Phase 4 Deliverable**: The home screen feels like a living, breathing intelligence surface. The business is communicating. It's a supercomputer.

---

## Files Summary

| Category | Count |
|----------|-------|
| New files | ~25 |
| Modified files | 3 |
| Deleted files | 0 |
| New dependencies | 1-2 (`@openuidev/react-lang`, maybe `@ai-sdk/openai`) |
| Routing changes | **0** |

---

## What This Means for the Audit Findings

| v2 Audit Issue | v3 Status |
|---|---|
| ❌ Routing refactor is bigger than described | **Eliminated.** No routing changes. |
| ❌ Cockpit widgets can't reuse Streamdown system | **Still true.** Cockpit widgets are standalone. But now clearer — they live in `components/cockpit/widgets/` and are never touched by Streamdown. |
| ⚠️ Zustand not installed | **Use React context** to match existing patterns. Create a `CockpitProvider` if needed. |
| ⚠️ Mobile popover/sheet should be Phase 2 | **Done.** Step 2.10. |
| ⚠️ Calendar install should be Phase 2 start | **Done.** Step 2.4. |
| ⚠️ OpenUI Dashboard uses renderNode not children | **Still applies.** Will be handled in Step 3.7 when defining libraries. |
| 🔍 Missing LLM provider setup | **Done.** Step 3.1. |
| 🔍 Missing onAction handler | **Addressed in Phase 3** when wiring renderers. |
| 🔍 Missing mock dimension data | **Done.** Step 1.6 creates mock vitals data. |

---

*v3 plan — 2026-04-19. Chat IS the cockpit. The business is alive.*
