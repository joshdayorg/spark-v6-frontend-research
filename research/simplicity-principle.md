# The Simplicity Principle: How Simple Can This Be?

> "Keep everything simple. Keep everything high-level. Don't over-specify."
> — JD, 2026-04-19

---

## The V3 plan was too complex

The V3 plan specified 25 new files, two OpenUI libraries, custom vitals components, a separate API route for cockpit generation, 40 build steps across 4 phases. That's us solving problems the model should solve.

The 5W1H primitives are as simple as it gets. The product's power comes from that simplicity, not from engineering around it.

## The leanest possible version

### What if the cockpit is just... a chat message?

The home screen today shows a greeting + chat input. What if we just auto-trigger a system turn on page load that says "communicate your state through 5W1H" — and the model responds with a full OpenUI-rendered cockpit?

**The cockpit is the first AI message in an ephemeral thread.**

No separate pipeline. No cockpit-specific components. No vitals API route. The same `<Renderer>` that renders every chat response renders the cockpit. The model picks components from the same built-in library.

### The entire pipeline

```
User opens / 
  → Convex query assembles context (name, time, dreams, memory, agents)
  → System prompt: "You are Spark. Communicate through 5W1H. Here's what you know: {context}"
  → LLM streams OpenUI Lang using the built-in component library
  → <Renderer> turns it into React components
  → Query() components fetch live data from Convex
  → User sees: a living cockpit
```

### New files needed

| File | What | Lines |
|------|------|-------|
| Context assembly function | Reads dreams, MEMORY.md, agent status from Convex → returns ~1K token snapshot | ~50 |
| Home screen upgrade | Auto-triggers cockpit generation on load, renders response via existing chat pipeline | ~30 |
| System prompt fragment | The 5W1H instruction + voice rules | ~20 |

**Total: ~100 lines of new code. Maybe 3 files.**

Everything else is the model's job.

### What we DON'T build

- ❌ Custom `StatusLine`, `MetricBadge`, `AlertSignal`, `PersonStatus`, `Sparkline` components — the built-in OpenUI library already has all of these
- ❌ Separate `vitalsLibrary` and `responseLibrary` — one library, the built-in one
- ❌ Separate `/api/cockpit` route — cockpit goes through the same chat/streaming pipeline
- ❌ `DimensionGrid.tsx`, `DimensionCard.tsx`, `DimensionSearch.tsx` as separate components — the model lays out the grid itself
- ❌ 10-step Phase 3 OpenUI integration — it's 2 steps: add the system prompt, render the response

### What we DO build

- ✅ A context snapshot function (Convex query/action)
- ✅ A system prompt that teaches the model to speak as the business through 5W1H
- ✅ A home screen that auto-triggers this on load and renders the response
- ✅ Tool definitions so Query() components can fetch live data from Convex

## The system prompt is the product

This is the key reframe. The product isn't components. The product is the prompt.

```
You are Spark, a sentient business intelligence system.
You communicate your state through six dimensions: Who, What, When, Where, Why, How.

Speak in first person as the business. Not metrics — meaning.
Not "3 PRs open" — "I need your review on 2 pull requests."
Not "99.2% uptime" — "I'm healthy. All systems running."

Here's what you know about this moment:
{contextSnapshot}

Generate a cockpit showing these six dimensions.
Use the component library to create cards, stats, charts, and signals.
Most urgent items first. Natural language. No jargon.
```

That's ~150 tokens of instruction. The rest is context + OpenUI's auto-generated component reference.

The model decides:
- How to lay out the 6 dimensions
- What signals to highlight in each
- What components to use (stat cards? tables? charts? just text?)
- What language to use
- What level of urgency to convey

We don't pre-decide any of that. We give it data and primitives and let it be intelligent.

## The interaction model simplifies too

### Click a signal → it becomes a chat message

OpenUI has `Action()` with `@ToAssistant()`. When the user clicks a signal in the cockpit, it sends that signal's text as a follow-up message in the chat. The model responds with deeper analysis.

No routing changes. No thread creation logic. No dimension search components. The cockpit is a chat message. Clicking a signal is sending a follow-up message.

### Refresh → regenerate

Pull down or click refresh → the cockpit regenerates. New context, new signals. The model decides what changed and what's important now.

### The search question

The V3 plan had elaborate per-dimension search with autocomplete. Simpler version: the chat input IS the search. Type "who" and the model focuses on the WHO dimension. Type "Sarah PRs" and the model answers about Sarah's PRs. No dimension-specific search UI needed. The model already understands the 5W1H framing from its system prompt.

If we want pill-based structured queries later, we can add that as a refinement. But the MVP doesn't need it.

## What about consistency?

The Skeleton/Flesh/Nerve model from the deep dive still applies, but simpler:

- **Skeleton**: the model will naturally produce a 6-card layout because the prompt asks for 6 dimensions. That's structural consistency.
- **Flesh**: the model picks different signals based on context. That's freshness.
- **Nerve**: Query() components fetch live data. That's real-time.

If the model generates inconsistent layouts (sometimes 4 cards, sometimes 8), we can constrain it in the prompt: "Always generate exactly 6 dimension cards." But try without the constraint first — the model might surprise us.

## Cost and performance

Same as the deep dive analysis:
- ~3K-5K input tokens (system prompt + context)
- ~400-800 output tokens (cockpit in OpenUI Lang)
- ~$0.01-0.03 per load (Sonnet/Haiku)
- ~1.5s streaming first paint, ~200ms cached
- Cache the OpenUI Lang string for 15 min

## The phased approach (simplified)

### Phase 1: Mock cockpit (2-3 days)
Hardcode an OpenUI Lang string that represents a 5W1H cockpit. Render it with `<Renderer>`. Get the visual right. No LLM yet.

### Phase 2: Live cockpit (2-3 days)
Replace hardcoded string with LLM generation. Add context assembly from Convex. Wire Query() components to Convex queries. The cockpit is now live.

### Phase 3: Polish (2-3 days)
Caching. Greeting intelligence. Signal click → chat continuation. Animations. Mobile.

**Total: ~1-2 weeks, not 4.**

## The danger we're avoiding

> "Too many rules, too many files, too many MD specs = brittle system."

Every custom component we write is a constraint on the model. Every spec file is a rule that might conflict with what the model naturally wants to do. Every separate pipeline is surface area to maintain.

The 5W1H primitives are powerful BECAUSE they're primitive. The model is powerful BECAUSE it's general. The data is powerful BECAUSE it's real.

Our job is to connect them with the minimum possible scaffolding. Then get out of the way.

## Open question

Is it actually *too* simple? Does the model reliably produce good 5W1H cockpits with just a system prompt and context? We won't know until we try. The fastest path to an answer is Phase 1: write the prompt, feed it context, see what comes out.

If the output is good → ship it.
If the output needs guardrails → add exactly the guardrails needed, one at a time.
Don't pre-build guardrails for problems we haven't seen yet.
