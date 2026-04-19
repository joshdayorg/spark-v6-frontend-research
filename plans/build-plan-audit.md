# Cockpit Build Plan — Audit Report

> Cross-reference of the step-by-step build plan against the actual `spark-v6-frontend-jd` codebase and OpenUI's real API.

---

## Overall Assessment

The plan is **directionally solid** but has several assumptions that need correcting before execution. The biggest issues are around OpenUI's actual API surface, the routing changes, and some underestimated complexity in the chat system interaction.

---

## Phase 1: Foundation

### ✅ Correct and Solid

- **Installing `@openuidev/react-lang`** — Confirmed. This is the right package. It exports `defineComponent`, `createLibrary`, and `Renderer`. No config needed.
- **Creating a `(cockpit)` route group** — Correct approach. Parallel to `(chat)` under `(app)`, shares the sidebar shell from `(app)/layout.tsx`.
- **DimensionPill using `popover.tsx`** — Good call. Popover is already in the project.
- **Zustand for state** — Fine choice, but note: Zustand is not currently installed. The project uses React context everywhere (e.g., `ChatThreadsProvider`, `ChatThreadScope`). Consider matching the existing pattern with a `CockpitProvider` context instead to stay consistent. If you want Zustand, add it to deps list.
- **Reusing `Greeting` animation pattern** — Smart. The `motion` entrance animations in `greeting.tsx` are a good template.
- **SuggestionRail reusing `suggested-actions.tsx` pattern** — Correct. Uses `ai-elements/suggestion.tsx` + motion animations.

### ⚠️ Needs Adjustment

**Step 1.3: Routing changes — more complex than described.**

The plan says "move chat to `/chat`" but the current routing is:
- `/` → `(app)/(chat)/page.tsx` (new chat)
- `/[threadId]` → `(app)/(chat)/[threadId]/page.tsx` (existing thread)

The `(chat)` route group is a **catch-all for the root**. Thread URLs are `/{threadId}` not `/chat/{threadId}`. The sidebar's `isChatPath()` function treats ANY path that isn't a known prefix (pulse, knowledge, agents, findings, settings) as a chat path.

To make cockpit the homepage:
- **Option A (simpler)**: Keep chat URLs as-is (`/[threadId]`). Add cockpit as a new route. But then `/` needs to decide: cockpit or new-chat? You'd need to move the new-chat page to an explicit `/chat` route.
- **Option B (cleaner)**: Move ALL chat routes under `/chat/` prefix: `/chat` (new thread), `/chat/[threadId]` (existing). Make `/` the cockpit.

**Recommendation**: Option B. But this means:
- Move `(chat)/page.tsx` → `src/app/(app)/chat/page.tsx`
- Move `(chat)/[threadId]/page.tsx` → `src/app/(app)/chat/[threadId]/page.tsx`
- Move `(chat)/layout.tsx` → `src/app/(app)/chat/layout.tsx`
- Update ALL thread links in sidebar (`/${thread.id}` → `/chat/${thread.id}`)
- Update `isChatPath()` in sidebar
- Update `useCreateThread` which does `router.push(\`/${threadId}\`)`
- This is a **real refactor**, not just adding a route. Budget it properly.

**Step 1.5: DimensionPill states — missing detail on the "AI-suggested" state.**

The plan mentions a "subtle pulse/shimmer when AI recommends filling this dimension" but doesn't specify when this fires. In Phase 1 there's no AI connected yet. Either:
- Remove this state from Phase 1 (add in Phase 2 when AI suggestions exist)
- Or implement it as a static demo animation

**Step 1.6: QuerySynthesisBar — clarify the dual-mode input.**

The plan says it "accepts free text typing" AND shows assembled dimension text. These are two different modes:
- Mode 1: Dimensions selected → bar shows read-only assembled text
- Mode 2: User clicks into bar → free text input

Need to spec which mode is default and how switching works. Recommendation: default to free text input, with dimension chips/tags inline (like a token input).

### ❌ Wrong or Won't Work

**Step 1.9: The plan says `synthesizedQuery` is "derived state" from dimension selections.** This is trivially correct for simple template assembly ("Show me [WHO]'s [WHAT] from [WHEN]") but the plan implies AI-powered synthesis later. The Phase 1 version should just be template-based string concatenation. Clarify this isn't calling an LLM in Phase 1.

### 🔍 Missing Items

- **`@openuidev/react-ui` CSS import**: If you use any prebuilt OpenUI UI components (even just for reference), you need `import '@openuidev/react-ui/components.css'`. The plan doesn't mention this. For Phase 1 you only need `react-lang` (no CSS), so this is fine for now but flag it for Phase 3.
- **New sidebar item**: The plan mentions adding Cockpit to the nav but doesn't specify the icon. Use `Gauge` from lucide-react (it's the closest to a "cockpit" metaphor). It's NOT currently imported in `sidebar.tsx`.
- **Thread URL migration**: If going with Option B routing, the `ThreadItem` component in `src/components/sidebar/thread-item.tsx` links to `/${thread.id}`. This needs updating to `/chat/${thread.id}`.

---

## Phase 2: Dimension Intelligence

### ✅ Correct and Solid

- **Using `command.tsx` (cmdk) for searchable pickers** — Already installed and used. Great pattern for WHO, WHAT, WHERE.
- **Using `button-group.tsx` for WHEN presets** — Already in the project. Good fit.
- **Keyboard shortcuts pattern** — `use-chat-shortcuts.ts` is a good reference for the implementation.

### ⚠️ Needs Adjustment

**Step 2.4: WhenDimension — Calendar component.**

The plan says "would need to add `calendar.tsx` from shadcn registry." But checking the current `components/ui/` directory: **calendar is NOT listed in the existing components.** Confirmed missing. The plan correctly flags this but buries it in Step 2.12. Move the `npx shadcn@latest add calendar` to the START of Phase 2, not the end.

**Step 2.8: Cross-dimension filtering — oversimplified.**

The plan says "when WHO changes, filter WHAT suggestions based on entity type." But WHERE does the suggestion data come from? In Phase 2 there's no backend connected. Options:
- Hardcoded suggestion sets per dimension (simple, works for demo)
- Local JSON data files with relationships
- Convex queries (requires backend work)

Recommendation: Hardcoded mock data for Phase 2. Add a `src/lib/mock-data/dimensions.ts` file with suggestion sets and cross-filtering logic.

**Step 2.9: Free text → dimension auto-population.**

The plan says "can be regex-based for Phase 2." This is fine but specify: WHO parsing should look for @mentions or capitalized names, WHEN should look for time words ("yesterday", "last week", "this month"), WHERE should match known source names. Define the regex patterns explicitly or the implementer will have to guess.

### 🔍 Missing Items

- **Mock data file**: Need `src/lib/mock-data/dimensions.ts` with:
  - People list for WHO
  - Action/topic list for WHAT
  - Time presets for WHEN
  - Source list for WHERE
  - Goal presets for WHY
  - Format options for HOW
  - Cross-filtering maps
- **Popover positioning**: On mobile, popovers may clip. The plan mentions using `sheet.tsx` for mobile in Phase 4 but this should be considered in Phase 2 architecture (conditional render popover vs sheet based on `use-mobile.ts`).

---

## Phase 3: Response Surface + OpenUI

### ✅ Correct and Solid

- **OpenUI `defineComponent` + `createLibrary` API** — Confirmed correct. The plan's code examples match the real API:
  ```tsx
  defineComponent({ name, description, props: z.object({...}), component: ({props}) => <JSX/> })
  createLibrary({ root: "Dashboard", components: [...] })
  ```
- **`<Renderer>` component** — Confirmed. Props are `response` (string), `library`, `isStreaming` (boolean), and optionally `onAction`.
- **Library `.prompt()` method** — Confirmed. Accepts `examples` and `additionalRules` options.
- **Vercel AI SDK integration** — Confirmed. OpenUI has an official `vercel-ai-chat` example using `useChat` + `<Renderer>`.

### ⚠️ Needs Adjustment

**Step 3.3: The `component` renderer receives `{ props }`, not spread props.**

The plan shows:
```tsx
component: ({ props }) => <CockpitDataTable {...props} />,
```
This is actually correct per the API — the renderer function receives an object with `props` and `renderNode`. But the plan's widget components need to accept a `props` object shape that matches the Zod schema exactly. Make sure the Zod schemas and widget component prop types are aligned.

**Step 3.4: System prompt generation — more involved than shown.**

The plan shows:
```tsx
const systemPrompt = cockpitLibrary.prompt({ examples, additionalRules });
```

This is correct, but the system prompt needs to be generated **server-side** (in the API route), not client-side. The plan's Step 3.6 does create an API route, but the prompt generation should be explicitly placed there. Also: the prompt is a string that includes the full OpenUI Lang syntax rules, component signatures, and examples. It can be large. Consider generating it at build time or caching it.

**Step 3.6: API route — the plan correctly notes this is the "first real LLM connection."**

This is a big deal. The current chat system is entirely mock-based. Adding a real API route means:
- Need to configure an LLM provider (OpenAI, Anthropic, etc.)
- Need API keys in environment variables
- Need to decide: does the cockpit API route share config with the (future) chat API route?
- The plan recommends standalone first (option a) — this is correct.

But add a step: **"Step 3.5.5: Configure LLM provider"** — install `@ai-sdk/openai` (or whichever provider), add `OPENAI_API_KEY` to `.env.local`, verify the AI SDK `streamText` works standalone before wiring to OpenUI.

**Step 3.8: Dashboard root component — the `children` pattern needs clarification.**

OpenUI's `createLibrary` `root` specifies which component wraps the top-level output. The plan defines a `Dashboard` component with `children: z.array(z.any())`. But in OpenUI Lang, children aren't passed as props — they're composed via the language's tree structure. The `component` renderer receives `renderNode` for rendering child nodes. Correct pattern:

```tsx
const Dashboard = defineComponent({
  name: "Dashboard",
  description: "Root layout for cockpit responses",
  props: z.object({
    title: z.string().optional(),
  }),
  component: ({ props, renderNode }) => (
    <div className="grid gap-4 p-4">
      {props.title && <h2>{props.title}</h2>}
      {/* renderNode handles child composition */}
    </div>
  ),
});
```

**Fix the plan's Dashboard component to use `renderNode` instead of `children` prop.**

### ❌ Wrong or Won't Work

**Step 3.1: "Leverage and extend the existing widget system."**

The existing widget system (`widget-renderer.tsx`) is tightly coupled to the **Streamdown custom tag system** (`streamdown-config.ts`). Widgets are injected via `<widget type="bar-chart" call="toolCallId" />` tags in markdown, and they depend on `MessagePartsContext` for tool call state.

The cockpit widgets **cannot reuse this injection mechanism**. They need to be standalone React components that accept props directly (for OpenUI). The plan should clarify:
- Cockpit widgets are **new standalone components** in `src/components/cockpit/widgets/`
- They can reuse the **visual design** from existing widgets (copy/adapt `bar-chart.tsx`, `stat-card.tsx`, etc.)
- But they must NOT depend on `MessagePartsContext`, `toolCallId`, or the Streamdown tag system
- The existing `src/lib/widgets/types.ts` type definitions can be reused/extended for the Zod schemas

### 🔍 Missing Items

- **LLM provider setup step**: Need `@ai-sdk/openai` or similar installed + env vars configured
- **`onAction` handler**: OpenUI's `<Renderer>` accepts `onAction` for interactive widget actions (button clicks, form submits). The plan doesn't mention wiring this. Important for interactive response widgets.
- **Error handling in Renderer**: What happens when the LLM generates invalid OpenUI Lang? The `<Renderer>` handles parsing errors gracefully but you should add a fallback UI.
- **Token budget**: OpenUI Lang system prompts can be large (includes all component signatures + rules + examples). With 8 widgets, estimate 2-4k tokens for the system prompt. Factor this into model context window planning.
- **Streaming protocol**: The plan says "stream responses via AI SDK with OpenUI Lang output" but doesn't specify: the LLM needs to be instructed to output OpenUI Lang (not markdown, not JSON). This is handled by the system prompt from `library.prompt()`, but verify the model actually follows it. May need few-shot examples in the prompt options.

---

## Phase 4: Polish & Integration

### ✅ Correct and Solid

- **Chat escalation using `useCreateThread`** — Correct hook exists. But note: the mock version creates a thread with `Date.now().toString(36)` as ID. This will change when Convex is wired.
- **Mobile using `use-mobile.ts`** — Correct hook exists.
- **Motion library for animations** — Already installed, already used in greeting/suggestions.
- **localStorage persistence matching existing pattern** — Smart. The chat provider already uses `CHAT_THREADS_STORAGE_KEY` pattern.

### ⚠️ Needs Adjustment

**Step 4.1: Chat escalation — context passing is underspecified.**

The plan says "pre-loaded with the cockpit query as the user message and cockpit response context as system context." But the current `createThread(prompt)` only accepts a string prompt. To pass cockpit context, you'd need to:
- Either: extend `createThread` to accept initial system context
- Or: create the thread with the synthesized query as the first user message, then include cockpit response summary as a follow-up system message
- The mock provider would need modification to support this

**Step 4.5: Mobile — popover → sheet swap.**

This should be built into the `DimensionPopover` shared wrapper from Phase 2, not deferred to Phase 4. Otherwise you'd need to refactor all 6 dimension components. Update Phase 2's `DimensionPopover.tsx` to conditionally render Popover (desktop) or Sheet (mobile) from the start.

### 🔍 Missing Items

- **Testing**: No mention of tests anywhere. At minimum: unit tests for dimension state management, integration test for the execute flow.
- **Loading/skeleton states**: The plan mentions shimmer for streaming but doesn't specify skeleton layouts for the response surface while waiting for first tokens.
- **Abort/cancel**: User should be able to cancel a cockpit query mid-stream. The AI SDK's `stop()` function handles this, but it needs to be wired to a cancel button on the response surface.

---

## Revised Step Ordering

Major ordering changes:

1. **Move calendar install to start of Phase 2** (not end)
2. **Move mobile popover/sheet conditional into Phase 2** (not Phase 4)
3. **Add LLM provider setup as explicit step in Phase 3** (before API route)
4. **Add mock data file for dimensions in Phase 2** (before cross-filtering)
5. **Routing refactor (chat → `/chat/*`) should be Step 1.2, before creating cockpit route** (it's a prerequisite)

---

## Risk Assessment

| Risk | Severity | Mitigation |
|------|----------|------------|
| Routing refactor breaks existing thread links | High | Do it first, test thoroughly before building cockpit |
| OpenUI system prompt too large for model context | Medium | Start with fewer widgets (3-4), add more incrementally |
| LLM doesn't reliably generate valid OpenUI Lang | Medium | Add few-shot examples to prompt options, test with multiple models |
| Mock chat system diverges from future Convex integration | Low | Build cockpit with clean interfaces that can swap backends |
| OpenUI library is relatively new (v0.5) | Medium | Pin version, have fallback rendering for parse failures |

---

## Summary

| Category | Count |
|----------|-------|
| ✅ Correct and solid | 12 items |
| ⚠️ Needs adjustment | 10 items |
| ❌ Wrong or won't work | 2 items |
| 🔍 Missing items | 10 items |

The plan is ~70% executable as-is. The two critical fixes are:
1. **The routing refactor is bigger than described** — budget it as a real task
2. **Cockpit widgets must be standalone** — can't reuse the Streamdown/tool-call widget injection system

Everything else is adjustments and additions, not rewrites.

---

*Audit completed 2026-04-18. Based on codebase review + OpenUI API verification via docs and Context7.*
