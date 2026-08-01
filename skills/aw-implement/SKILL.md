---
name: aw-implement
description: Build the work — write the code for a feature or fix (from a spec, tickets, or a described task), test-first at the agreed seams, then review and commit. Use when the next step is writing code, or the user says "implement" / "implementá" / "buildeá". Declines large or unplanned work, routing it back to planning.
---

# aw-implement

Build the work — the tail of the harness, where a plan becomes code. It runs two ways from
one skill: **standalone** on a small, clear task, or as the last step of the heavy flow
(`aw-grill-plan` → `aw-write-spec` → `aw-slice-plan` → here). The skill itself decides which
it is.

## Guard — two gates, from the build side

Read the work before writing a line. `aw-write-spec` gates the same task from the spec side
with two tests — **magnitude** (small vs large) and **ripeness** (settled vs open decision).
Apply those same two here, and let its guard own their definitions so the two never drift.

- **Small and settled — build now.** Go straight to the steps: no spec, no tickets, no
  ceremony.
- **Large, or any blocking decision still open — plan first.** Don't improvise it: route
  back — `aw-grill-plan` if a decision is open, `aw-write-spec` / `aw-slice-plan` if it's
  settled but unsliced. Building large or unripe work is the failure this gate exists to stop.

Done when the task is confirmed small and settled, or handed back to the planning flow.

## Steps

### 1. Gather the work

If a spec or tickets were produced, `fetch` them through
`skills/_shared/artifact-convention.md` — the full spec body, or the ticket and its blocking
edges. If the task was described directly, the conversation is the brief. Any fact you can
find by exploring the codebase, look up rather than ask. Done when what to build, and the
seams it's tested at, are in front of you.

### 2. Choose where to build

The **current branch** is the floor — always available. A large or risky change, or one of
several tickets run in parallel, wants isolation instead: a dedicated worktree spawned via
`aw-spawn-worktree`, falling back to the current branch when one can't be spawned. Done when
the branch or worktree to build in is named.

### 3. Build test-first

Drive each vertical slice through `aw-tdd`: red, green, refactor. Done when every behaviour
the brief describes is covered by a passing test at an agreed seam.

### 4. Review before you commit

Hand the finished change to `aw-review-work` as an explicit review request — it only reviews
and reports; you fold back what it surfaces, applying any fixes it flags (re-entering
`aw-tdd` for behavioural ones). Done when the review passes or its findings are addressed.

### 5. Commit — only when asked

Commit to the current branch with a conventional-commit message, no attribution — but only
when the user has asked to commit. Otherwise stop after the review and report the change,
ready for them to commit. Done when the work is committed, or handed back ready to commit.

## Attribution

Adapted for this workspace from **Matt Pocock's `implement`** (github.com/mattpocock/skills,
Apache-2.0). Changes: switched to model-invocation and added a **two-gate guard** (magnitude
+ ripeness, whose definitions `aw-write-spec` owns) so the harness itself decides whether to
build or route back to planning; `/tdd` and `/code-review` become the harness's `aw-tdd` and
`aw-review-work` (which reviews only); work is `fetch`ed through the artifact-convention and
isolated in a worktree where the workspace can spawn one (the seam a future execution
substrate plugs into), falling back to the current branch; the commit is gated on the user
having asked. The spec-or-tickets input and test-first-at-seams spine
are preserved.
