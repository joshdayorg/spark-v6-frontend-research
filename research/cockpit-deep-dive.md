# Deep Research: The Simplest Path to a 5W1H Cockpit

> Research for `joshdayorg/spark-v6-frontend-research`
> Author: Spark UI agent
> Date: 2026-04-19

---

## Question 1: What is the bare minimum interface?

### The simplest possible version

Yes — it really is this simple at the core:

```
User identity + Context → System prompt + OpenUI component library → LLM → OpenUI Lang → Renderer → Cockpit
```

**One API call. One render.** The user loads the page, an API route sends context to Anthropic/OpenAI with the OpenUI system prompt, the LLM streams back OpenUI Lang, the `<Renderer>` turns it into React components. Done.

Here's what the actual code looks like:

```typescript
// src/app/api/cockpit/route.ts
import { generatePrompt } from "@openuidev/lang-core";
import componentSpec from "./generated/cockpit-spec.json";
import Anthropic from "@anthropic-ai/sdk";

export async function POST(req: Request) {
  const { userContext } = await req.json();

  const systemPrompt = generatePrompt({
    ...componentSpec,
    preamble: `You are the intelligence layer for a business called Spark.
You speak in first person as the business. You are communicating your state to ${userContext.name}.
Generate a 5W1H cockpit showing WHO, WHAT, WHEN, WHERE, WHY, HOW.
Each dimension shows 2-4 signals. Use natural language, not metrics.
Here is the business context:\n${userContext.memory}`,
    toolCalls: true,
    tools: cockpitTools,
  });

  const stream = await anthropic.messages.stream({
    model: "claude-sonnet-4-5-20250514",
    system: systemPrompt,
    messages: [{ role: "user", content: "Show me the cockpit." }],
    max_tokens: 2000,
  });

  return new Response(stream.toReadableStream());
}
```

```tsx
// src/components/cockpit/HomeScreen.tsx
import { Renderer } from "@openuidev/react-lang";
import { cockpitLibrary } from "./cockpit-library";

export function HomeScreen({ stream, isStreaming }) {
  return (
    <Renderer
      library={cockpitLibrary}
      response={stream}
      isStreaming={isStreaming}
      toolProvider={cockpitToolProvider}
    />
  );
}
```

That's the whole thing. ~40 lines of real code for the pipeline.

### Does it change every time you load it?

**Yes — and that's the point.** Each load generates a fresh cockpit based on current context. The business is communicating what matters RIGHT NOW, not showing a cached dashboard.

But this creates a real tension (see Question 6). The answer isn't "always regenerate" or "always cache." It's a hybrid.

### The right caching/persistence model

Three layers:

| Layer | What | Cache strategy |
|-------|------|---------------|
| **Context** (input) | USER.md, MEMORY.md, dream reports, agent state | Read once on page load. Invalidate when memory files change. |
| **OpenUI Lang** (output) | The generated cockpit code | Cache for 5-15 minutes. Serve cached on repeat loads. Regenerate on explicit refresh or context change. |
| **Live data** (runtime) | Query results from `Query()` components | Fresh on every load — OpenUI runtime fetches these independently of the LLM. |

**Key insight from OpenUI's architecture:** The LLM generates the *wiring* (which queries to run, which components to show, how to lay them out). The *data* flows through `Query()` components that call your tools directly, without the LLM. So you can cache the wiring and still get fresh data.

```
Cache the LAYOUT (LLM output) — regenerate every 15 min or on context change
Always fresh DATA (Query results) — fetched by runtime on every load
```

This means the cockpit *structure* might be the same across loads (same 6 cards, same signal types), but the *numbers and text inside* are always live.

---

## Question 2: The minimum viable pipeline

### From page load to rendered cockpit:

```
1. User navigates to /
2. Client: read userId from Clerk session
3. Client: call useQuery(api.cockpit.getContext, { userId })
   → Convex returns: { name, timezone, dreamReports, memorySnapshot, activeAgents }
4. Client: POST /api/cockpit with context
   → If cached OpenUI Lang exists and is <15 min old: return cached
   → Else: generate new via LLM
5. Client: stream response through <Renderer library={cockpitLibrary}>
6. Renderer: parses OpenUI Lang, renders components
7. Query() components: fire tool calls for live data (agent status, system health)
8. User sees: 5W1H cockpit with live intelligence
```

**Time budget:**
- Step 3: ~50ms (Convex subscription, likely already warm)
- Step 4 (cached): ~10ms (serve from Redis/memory)
- Step 4 (regenerate): ~2-4 seconds (LLM streaming)
- Step 5-7: ~100ms (OpenUI parsing + initial render + tool calls)

**First meaningful paint: ~200ms (cached) or ~1.5s (streaming, progressive render)**

### The minimum viable data flow:

```
Convex (context) ──→ API route ──→ LLM ──→ OpenUI Lang ──→ Renderer
                                                              ↓
                                                         Query("agent_status") ──→ Convex
                                                         Query("system_health") ──→ Convex
                                                         Query("recent_activity") ──→ Convex
```

The LLM decides WHAT to show (layout, signal selection, language). Convex provides HOW to show it (live data). Two separate concerns.

### What you need to build:

| Component | What it does | Exists? |
|-----------|-------------|---------|
| Convex function: `cockpit.getContext` | Assembles user context from memory/agents | No — new |
| API route: `/api/cockpit` | Sends context to LLM, returns OpenUI Lang | No — new |
| OpenUI library: `cockpitLibrary` | Defines the 5W1H component vocabulary | No — new |
| Tool provider: `cockpitToolProvider` | Maps Query names to Convex queries | No — new |
| React component: `<HomeScreen>` | Wires it all together | Exists — upgrade |

**5 things. That's it.**

---

## Question 3: How OpenUI actually works in practice

### The system prompt

When you call `cockpitLibrary.prompt(options)`, OpenUI generates a system prompt that includes:

1. **Syntax rules** — how to write OpenUI Lang (variable declarations, component instantiation, expressions)
2. **Component signatures** — every component in your library with its Zod-validated props
3. **Built-in functions** — `@Count`, `@Filter`, `@Sort`, `@Each` for data manipulation
4. **Query/Mutation workflow** — how to use `Query("tool_name", args, defaults)` and `Mutation`
5. **Reactive binding rules** — how `$variables` create two-way bindings
6. **Your custom preamble** — "You are Spark, a sentient business..."
7. **Your examples** — sample OpenUI Lang showing desired output patterns
8. **Your additional rules** — "Always use natural language, not metric-speak"

### How big is the prompt?

Based on the OpenUI benchmarks:

| Library size | System prompt tokens | Notes |
|-------------|---------------------|-------|
| Default `openuiLibrary` (40+ components) | ~3,000-4,000 tokens | Full suite with charts, forms, tables |
| Custom `cockpitLibrary` (~15 components) | ~1,500-2,000 tokens | Smaller, focused library = smaller prompt |
| + tools (5-6 Convex queries) | +300-500 tokens | Tool descriptions |
| + preamble + context | +500-2,000 tokens | Depends on how much memory we inject |
| **Total** | **~2,500-5,000 tokens input** | |

The LLM output (the OpenUI Lang cockpit code) is typically 300-800 tokens for a dashboard layout.

### Token cost per page load

| Model | Input cost | Output cost | **Per cockpit load** |
|-------|-----------|------------|---------------------|
| Claude Sonnet 4 | $3/1M in, $15/1M out | | ~$0.015-0.025 |
| Claude Haiku 3.5 | $0.80/1M in, $4/1M out | | ~$0.004-0.007 |
| GPT-5.4-mini | ~$0.40/1M in, $1.60/1M out | | ~$0.002-0.005 |

**With 15-minute caching:** If a user loads the page 10 times in an hour, only 4 LLM calls happen. Cost: ~$0.02-0.10/hour per user (Sonnet) or ~$0.01-0.03/hour (Haiku/mini).

**At scale:** 100 active users × 8 hours × $0.05/hour = **$40/day with Sonnet**. Acceptable for a premium product.

### Latency

| Phase | Time |
|-------|------|
| Context assembly (Convex) | 50ms |
| LLM time to first token | 300-800ms |
| Full generation (streaming) | 2-4 seconds |
| OpenUI parse + initial render | 50-100ms |
| Query() tool calls (parallel) | 100-300ms |
| **Time to first meaningful paint** | **~500ms (cached) / ~1.5s (streaming)** |

With streaming, users see the cockpit *materializing* — cards appear one by one. This actually feels more "alive" than an instant load.

### Can you cache the generated UI?

**Yes, and OpenUI is designed for this.** The generated OpenUI Lang is just a string. You can:

1. Store it in Convex (per user, with timestamp)
2. Serve the cached string to the `<Renderer>`
3. The Renderer parses it and fires `Query()` calls for fresh data
4. Result: cached layout + live data = fast load + fresh intelligence

```typescript
// Cache the OpenUI Lang, not the rendered HTML
const cached = await ctx.db.query("cockpitCache")
  .filter(q => q.eq(q.field("userId"), userId))
  .first();

if (cached && Date.now() - cached.generatedAt < 15 * 60 * 1000) {
  return cached.openuiLang; // serve cached layout
}
// else regenerate via LLM
```

### What about incremental editing?

OpenUI supports `editMode: true` where the LLM outputs only changed statements. This is powerful for the cockpit:

1. First load: LLM generates full cockpit (~400 tokens)
2. Context changes (new PR opened, agent finished): send the *delta* to LLM
3. LLM outputs a patch (~60 tokens): "update the WHAT card's second signal"
4. Parser merges the patch into the existing code

**This reduces the regeneration cost by 85%.** Instead of regenerating the whole cockpit when something changes, you patch it.

### Query() and Mutation() — the key to live data without re-prompting

This is the most important OpenUI concept for the cockpit:

```
// LLM generates this ONCE:
agentStatus = Query("get_agent_status", {}, {agents: []})
recentPRs = Query("get_recent_prs", {limit: 5}, {prs: []})
systemHealth = Query("get_system_health", {}, {status: "unknown"})

// The Renderer calls these tools DIRECTLY:
// get_agent_status → Convex query → returns live data
// get_recent_prs → Convex query → returns live data  
// get_system_health → Convex query → returns live data
```

**Zero tokens consumed for data refresh.** The LLM decides *what data to fetch*. The runtime fetches it. If you add a `$timeRange` variable bound to a date picker, changing the range re-fetches all dependent queries automatically — still zero LLM involvement.

---

## Question 4: What context feeds the AI?

### Minimum viable context (~1,000 tokens)

```markdown
## User
Name: JD
Timezone: America/Los_Angeles
Current time: Sunday April 19 2026, 8:50 AM PDT

## Active Agents
main, twitter, forge, capital, observatory, spark-ui, luna, boost
(8 active, 4 idle, 16 dormant)

## Latest Dream Summary
[Inject most recent dream report — spark.md or team.md, whichever is freshest]

## Current Focus
[Inject last 3 lines of main workspace MEMORY.md — the "what's happening now" section]
```

This is enough for the LLM to generate a meaningful cockpit. The LLM uses this to decide:
- WHO: which people/agents to highlight
- WHAT: which activities/signals to surface
- WHEN: time-aware greeting + upcoming events
- WHY: current goals from MEMORY.md
- HOW: process state from dream reports

### Ideal context (~3,000 tokens)

Add:
- Last 2 daily notes (recent activity)
- All 4 dream reports (capital, content, spark, team) — condensed
- Agent status (which are running, which errored)
- System health (if available)

### Context assembly strategy

**Don't send raw files.** Send a *compiled context snapshot* — a Convex function that reads the relevant files, extracts the key sections, and returns a structured summary. This:
1. Keeps token count predictable
2. Avoids sending stale/irrelevant content
3. Can be cached (context changes slowly — maybe every 30 min)

```typescript
// convex/cockpit.ts
export const getContext = query({
  args: { userId: v.string() },
  handler: async (ctx, args) => {
    const user = await ctx.db.query("users").filter(...).first();
    const dreams = await loadLatestDreams(ctx); // from memory/dream/*.md
    const memory = await loadMemorySnapshot(ctx); // last 500 chars of MEMORY.md
    const agents = await loadAgentStatus(ctx);
    
    return {
      name: user.name,
      timezone: user.timezone,
      dreams: dreams.map(d => d.summary), // pre-condensed
      memory: memory.currentFocus,
      agents: agents.map(a => ({ id: a.id, status: a.status, lastActive: a.lastActive })),
    };
  },
});
```

### The phased approach

| Phase | Context source | How |
|-------|---------------|-----|
| **Phase 1 (mock)** | Hardcoded mock context | JSON file with fake user/agents/dreams |
| **Phase 2** | Real USER.md + MEMORY.md | Read from filesystem via Convex action |
| **Phase 3** | + Dream reports + agent status | Full compiled context |
| **Phase 4** | + Real-time signals (GitHub, Slack) | Connected data sources via tools |

---

## Question 5: Prior art on generative dashboards

### Who else is doing "AI generates the entire dashboard"?

| Product | Approach | What works | What fails |
|---------|----------|-----------|------------|
| **OpenUI Demo** (GitHub) | User types prompt → full dashboard generated | Streaming feels good. Incremental edits work. Tool-connected queries run live. | No persistence. Refresh = gone. No concept of "your dashboard." |
| **GlacierHub** | "Speak agents into existence" → agent swarms build dashboard | Marketplace of pre-built agents. Agents monitor anomalies. | Requires describing what you want. Not proactive. |
| **Knowi Agentic BI** | 15 AI agents handle analytics workflow | Direct data source connection. No warehouse needed. | Still chat-first ("ask a question, get a dashboard"). |
| **ICS A2UI (Flutter)** | 56-component catalog. Agent composes UI from catalog. | Constrained generation = consistent output. Production-ready components. | Requires explicit user request. Not ambient/proactive. |
| **Vercel AI SDK RSC** | `createStreamableUI()` → server pushes React components | Progressive streaming. AI State/UI State split is elegant. | Experimental. RSC complexity. Not designed for persistent dashboards. |
| **MindPalace** | "Your Business is Not 20 Dashboards. It's One Living Map." | North Star metric → everything connects. | Appears to be static once configured. Not generative. |

### Key patterns that work:

1. **Constrained generation** (ICS): The agent picks from a catalog, doesn't generate arbitrary UI. This keeps output predictable. OpenUI's library system does exactly this.

2. **Generate once, execute independently** (OpenUI): The LLM creates the wiring, then Query/Mutation components run without the LLM. This is the right architecture for a cockpit — generate the layout, let live data flow through it.

3. **AI State / UI State split** (Vercel): Keep a serializable representation (AI State) alongside the rendered components (UI State). The AI State can be persisted and sent back to the LLM. For us: the OpenUI Lang string IS the AI State.

4. **Incremental editing** (OpenUI): When context changes, patch the existing dashboard instead of regenerating. 85% token savings. Preserves user's mental model.

### What fails:

1. **Full regeneration on every interaction** — expensive, slow, inconsistent
2. **No persistence** — user refreshes, dashboard is gone
3. **Chat-first** — requiring the user to describe what they want defeats the "proactive intelligence" premise
4. **Arbitrary generation** — letting the LLM output raw HTML/CSS leads to inconsistent, broken UIs

---

## Question 6: Consistency vs. freshness

This is the hardest design question. Here's my analysis:

### The spectrum

```
FULLY STATIC                                              FULLY DYNAMIC
Traditional dashboard                                     New cockpit on every load
"Same widgets every time"                                 "Different every time"
Predictable, boring                                       Alive, chaotic
```

### What users actually want

Based on the research, users want:

1. **Predictable structure** — "I know WHERE to look for team status (top-left card)"
2. **Fresh content** — "The specific signals change based on what's happening"
3. **Surprise when warranted** — "If something unusual happens, show me something I didn't expect"

This maps to a three-layer model:

### The recommended architecture: Skeleton + Flesh + Nerve

| Layer | What | Persistence | LLM involvement |
|-------|------|------------|----------------|
| **Skeleton** (layout) | 2×3 grid, 6 dimension cards, greeting area, chat input | Permanent. Never changes. Pure React. | None |
| **Flesh** (signals) | 2-4 signals per card, generated by LLM | Cached 15 min. Regenerated on context change. | Yes — generates OpenUI Lang for signal content |
| **Nerve** (live data) | Numbers, statuses, sparklines inside signals | Always fresh. Query() components fetch from Convex. | None — runtime only |

**What this means in practice:**

- The **6 cards are always there**, always in the same position. WHO is always top-left. WHAT is always top-right. The user builds muscle memory.
- The **signals within each card change** based on what's happening. Monday morning: WHAT shows "weekend recap." During an incident: WHAT shows "3 alerts active." The LLM decides what's most important to surface.
- The **data within signals is always live**. "Sarah deployed 3x today" → the "3" is a Query() result that updates in real-time.

### How to handle the "where did my widget go?" problem

1. **Pin important signals.** If the user clicks a star/pin on a signal, it persists across regenerations. The LLM is told "these signals are pinned, include them."
2. **Rank, don't replace.** Instead of generating completely different signals each time, have the LLM rank ALL available signals and show the top N. The pool of possible signals is fixed (defined by your tools); the ordering changes.
3. **Animate transitions.** When signals change between loads, use shared-element transitions (Framer Motion `layoutId`). The signal doesn't "disappear" — it animates to its new position or fades out while the replacement fades in.
4. **Show history.** "This card showed X last time. Now it shows Y because [reason]." The LLM can include a tiny `meta` field explaining why it chose these signals.

### The simplest implementation

For Phase 1, don't even solve this problem. Use mock data → signals are static. The consistency/freshness question only matters when the LLM is generating real signals (Phase 3).

For Phase 3, start with: **cache OpenUI Lang for 15 minutes. Regenerate on explicit refresh.** See if users complain about staleness or inconsistency. Tune from there.

---

## Synthesis: The Simplest Elegant Path

### Architecture decision

```
┌─────────────────────────────────────────────────┐
│                  HomeScreen.tsx                   │
│  ┌────────────────────────────────────────────┐ │
│  │           Greeting (React, time-aware)       │ │
│  └────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────┐ │
│  │           DimensionGrid (React, fixed 2×3)   │ │
│  │  ┌──────┐ ┌──────┐ ┌──────┐                 │ │
│  │  │ WHO  │ │ WHAT │ │ WHEN │  ← card shells  │ │
│  │  │      │ │      │ │      │    are React     │ │
│  │  └──────┘ └──────┘ └──────┘                  │ │
│  │  ┌──────┐ ┌──────┐ ┌──────┐                 │ │
│  │  │WHERE │ │ WHY  │ │ HOW  │  ← signal       │ │
│  │  │      │ │      │ │      │    content is    │ │
│  │  └──────┘ └──────┘ └──────┘    OpenUI        │ │
│  └────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────┐ │
│  │           MultimodalInput (existing)         │ │
│  └────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────┘
```

**The skeleton is React. The flesh is OpenUI. The data is Convex.**

### The 3 things you need to decide

1. **Which model for cockpit generation?** Sonnet for quality, Haiku for speed/cost. My recommendation: **Haiku for signals** (fast, cheap, good enough for 2-line signal cards), **Sonnet for chat responses** (richer, deeper). Two different models for two different jobs.

2. **Cache invalidation strategy?** Start with time-based (15 min TTL). Graduate to event-based (Convex subscription triggers regen when dream reports update or agent status changes).

3. **How much context to inject?** Start minimal (1,000 tokens: name + time + latest dream + top 3 MEMORY.md lines). Add more only when signals feel generic.

### What this means for the V3 plan

The V3 plan is 80% right. The refinements based on this research:

- **Phase 1 stays mock** — correct. Don't touch OpenUI until layout is proven.
- **Phase 3 (OpenUI) is simpler than the plan implies** — it's one API route, one library, one Renderer. Not 10 steps. Maybe 4.
- **The toolProvider is the bridge to Convex** — this is the most important new concept. Each `Query("get_agent_status")` maps to a Convex query. This is how live data flows without the LLM.
- **Caching goes in the API route, not the component** — check for cached OpenUI Lang before calling the LLM. Serve cached layout + let Query() components fetch fresh data.
- **The greeting should be a SEPARATE LLM call** (lighter, faster, cacheable for 5 min). Don't bundle it with the cockpit generation.

---

## Open questions for JD

1. **Do we want the cockpit to work offline/pre-LLM?** If the LLM is down, should the cockpit show a static fallback? Or is LLM-generated-or-nothing acceptable?

2. **Should signals link to existing Spark pages or always start new threads?** The plan says "click signal → create thread." But some signals might link to existing Observatory/Knowledge pages instead.

3. **How real-time is real-time?** Convex subscriptions can push updates to the UI in ~50ms. Do we want signals to update WHILE the user is looking at the cockpit? Or only on page load?

4. **Multi-org support?** If one user belongs to multiple orgs, does the cockpit show signals for all of them, or is it scoped to the current org?

5. **Mobile-first or desktop-first?** The 2×3 grid works on desktop. On mobile, it becomes a scrollable list of 6 cards. Should mobile show fewer signals per card, or the same density with scrolling?
