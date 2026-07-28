---
name: aw-slice-plan
description: Slice an approved plan or spec into tracer-bullet tickets. Use when the user has a plan, spec, or decision ready and wants it broken into executable tickets to build.
---

A **tracer bullet** is a thin path fired end to end — one ticket you can grab and finish
in a single sitting. Slicing a plan into tracer bullets, each declaring its **blocking
edges**, is what turns an airtight spec into executable work.

## Steps

### 1. Gather context

Work from the plan or spec already in the conversation. If the spec was published out of
conversation — a fresh agent, a prior session — `fetch` the `spec` kind by its feature slug
through `skills/_shared/artifact-convention.md` first. Done when the full plan or spec is in
front of you.

### 2. Explore the codebase, if you haven't

Read enough of the code to slice against its real shape — so ticket titles use the
project's own vocabulary and slices respect the seams already there. Look for
**prefactoring** that makes the work easier: *make the change easy, then make the easy
change* — any prefactoring becomes the first slice. Done when you can name tickets in the
code's own vocabulary and have decided whether prefactoring is needed.

### 3. Draft the vertical slices

Break the work into **tracer bullet** tickets.

<vertical-slice-rules>

- Each slice cuts a narrow but complete path through every layer the change touches (e.g. schema, API, UI, tests) — vertical, end to end
- A completed slice is demoable or verifiable on its own
- Each slice is sized to fit in a single fresh context window

</vertical-slice-rules>

Give each ticket its **blocking edges** — the tickets that must finish before it can start.
A ticket with no blockers can start immediately.

**Wide refactors are the exception.** A **wide refactor** — rename a column, retype a shared
symbol — has a **blast radius** that breaks thousands of call sites at once, so no vertical
slice lands green. Sequence it **expand–contract** instead: expand (add the new form beside
the old, nothing breaks), migrate the call sites in batches sized by blast radius (each
batch a ticket blocked by the expand), then contract (delete the old form, in a ticket
blocked by every batch). Keep CI green batch to batch where each batch stands alone; where
it can't, the batches share an integration branch that all block one final
integrate-and-verify ticket, and green is promised only there.

Done when every slice obeys the rules above (expand–contract batches excepted) and its
blocking edges are drawn.

### 4. Quiz the user

Present the breakdown as a numbered list. Per ticket show **Title**, **Blocked by**, and
**What it delivers** — the end-to-end behaviour it makes work. Then ask:

- Is the granularity right — too coarse, too fine?
- Are the blocking edges correct — does each ticket depend only on tickets that genuinely
  gate it?
- Should any tickets merge or split?

Iterate until the user approves; publish only once they do.

### 5. Publish the tickets

`publish` each approved ticket through `skills/_shared/artifact-convention.md`, in
dependency order (blockers first) so each ticket's edges can name real ones. Work the
**frontier** — any ticket whose blockers are all done; for a linear chain that is top to
bottom. Fill the template per ticket; the convention decides where it lands and how its
edges are drawn. Done when every ticket the user approved in step 4 has a published artifact.

<ticket-template>

**What to build:** the end-to-end behaviour this ticket makes work, from the user's
perspective.

**Blocked by:** the tickets that gate this one, or "None — can start immediately".

- [ ] Acceptance criterion 1
- [ ] Acceptance criterion 2

</ticket-template>

Tickets describe behaviour and acceptance criteria. A file path or code snippet goes stale
fast — the one exception is a prototype snippet that pins a decision prose can't (a schema, a
type shape, a state machine); inline just the decision-rich part and say it came from a
prototype.

## Attribution

Adapted for this workspace from **Matt Pocock's `to-tickets`** (github.com/mattpocock/skills,
Apache-2.0). Changes: renamed to the `aw-<verb>-<noun>` convention; `publish` and `fetch`
route through `skills/_shared/artifact-convention.md` instead of a `setup`-bound tracker
config, so the skill never names a tracker; Matt's two ticket templates (local vs. issue)
collapse into one, since the convention — not the skill — maps a ticket's edges onto the
backend; dropped the triage-label vocabulary and the OpenAI agent wiring. The vertical-slice
rules, blocking edges, expand–contract exception, and frontier are preserved from the source.
