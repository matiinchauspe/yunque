---
name: yun-write-spec
description: Write the spec — consolidate a large task's settled decisions into one durable document before it is sliced into tickets. Use when the decisions are settled and the user wants them written up, or says "write the spec" / "escribí el spec". Declines small fixes that need no spec.
---

# yun-write-spec

A spec is written by **synthesis, not interview**. Everything already decided — in this
conversation and in the decisions `yun-grill-plan` persisted — gets consolidated into one
durable document. You ask the user **nothing**: if a decision is still open, you don't
write, you surface it.

## Guard — two gates before you synthesize

A spec is a commitment artifact. Pass both gates before writing a line.

- **Magnitude.** Does this task warrant a spec at all? A task that crosses more than one
  component, or is too big for a single session, is **large** and earns a spec. A small fix
  — one bug, a change to a single component within a single session — does not: say so and
  stop, the work goes straight to implementation.
- **Ripeness.** Are the decisions settled? `recall` prior decisions on the task's keywords
  (`skills/_shared/memory-convention.md`). A blocking decision still open means the
  conversation is not **ripe** — surface those decisions and hand back to `yun-grill-plan`
  rather than write a half-spec. Where recall is unavailable or returns nothing, judge
  ripeness from the conversation alone — an empty recall is not proof of ripeness, so don't
  bounce a task back to `yun-grill-plan` on missing memory.

Done when the task is confirmed large **and** every blocking decision is settled.

## Steps

### 1. Gather context — no interview

Pull the whole picture together: the conversation, plus the decisions `yun-grill-plan`
persisted (`recall` via `skills/_shared/memory-convention.md`). When the task was charted by
`yun-chart-course`, also `fetch` the `map` kind (`skills/_shared/artifact-convention.md`) and
follow its **Decisions so far** links to the resolved decision tickets — the map is an index,
so the decisions themselves live in the tickets, not the one-line gists. Any fact you can find
by exploring the codebase, look up rather than ask. Synthesis only — the user answers nothing.
Done when every settled decision is in front of you.

### 2. Sketch the testing seams

Decide **where the feature gets tested** — its **testing seams** — before writing. Prefer a
seam that already exists to a new one. Use the **highest** seam that still covers the
behaviour, and the **fewest** — the ideal number is one. These become the Testing Decisions section,
and they are the anchor `yun-slice-plan`'s tracer bullets verify against. Done when every
behaviour the spec will describe has a named seam.

### 3. Write the spec

Fill the template by synthesis. Implementation Decisions is where grill-plan's persisted
decisions land consolidated; Testing Decisions carries the seams from step 2; Further Notes
holds only non-blocking residuals — anything blocking was caught by the guard. Every other
section — Problem, Solution, User Stories, Out of Scope — traces to the conversation and the
persisted decisions directly.

<spec-template>

# <Feature title>

**Problem** — what's missing or broken, from the user's perspective.

**Solution** — the approach chosen to solve it.

**User Stories** — the behaviours this delivers, each from the user's perspective.

**Implementation Decisions** — the settled technical decisions, consolidated.

**Testing Decisions** — the seam(s) the feature is tested at, from step 2.

**Out of Scope** — the non-goals, named explicitly.

**Further Notes** — residual, non-blocking context.

</spec-template>

Done when every section traces to something already decided — no invention.

### 4. Publish the spec

`publish` the spec as the `spec` kind through `skills/_shared/artifact-convention.md`; the
convention decides where it lands and under which key. `yun-slice-plan` will `fetch` it from
there. Done when the spec is published and its location is known.

## Attribution

Adapted for this workspace from **Matt Pocock's `to-spec`** (github.com/mattpocock/skills,
Apache-2.0). Changes: added the two-gate guard (magnitude + ripeness) so it fires
autonomously without producing a premature spec; `recall` and `publish` route through the
memory- and artifact-conventions instead of a `setup`-bound tracker, so the skill never
names a tracker; the spec publishes as the `spec` artifact kind alongside `yun-slice-plan`'s
tickets. The highest/fewest-seam testing framing and the seven-section template are
preserved from the source.
