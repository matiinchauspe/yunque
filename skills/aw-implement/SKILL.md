---
name: aw-implement
description: Build the work — write the code for a feature or fix (from a spec, tickets, or a described task), test-first at the agreed seams, then review and land it when asked. Use when the next step is writing code, or the user says "implement" / "implementá" / "buildeá". Declines large or unplanned work, routing it back to planning.
---

# aw-implement

Build the work — where a plan becomes code. It runs two ways from
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
- **One run, one ticket.** aw-implement builds a single ticket or task. A multi-ticket plan is
  a frontier to walk — the orchestrator's job, not this skill's: route it back to planning
  rather than selecting one and inventorying the rest.

Done when the task is confirmed small and settled, or handed back to the planning flow.

## Steps

### 1. Gather the work

If a spec or tickets were produced, `fetch` them through
`skills/_shared/artifact-convention.md` — the full spec body, or the one ticket you'll run. If
the task was described directly, the conversation is the brief. Any fact you can find by
exploring the codebase, look up rather than ask. Done when what to build is in front of you,
with the test seams for it where any exist.

### 2. Choose where the run happens

`skills/_shared/execution-convention.md` governs the isolated run of a handed ticket — takeable,
unattended, branch-producing. What picks the mode is that provenance: a task you build directly
and attend yourself is none of those, so it stays outside the contract
and builds on the current branch, single-writer — its commit a local landing the human driving it
owns, not a fold. A ticket handed to you to run, or a large or risky change worth isolating, calls
`isolate` through the contract, which resolves the run's workspace down its ladder (a sandbox,
else an isolated worktree via `aw-spawn-worktree`, else the host tree's single-writer floor),
every tier on its own named branch. Done when the workspace is chosen: the current branch for a
task you build in place, or an isolated workspace on its own named branch.

### 3. Converge on done

Loop the build until it reaches its done-signal: wherever the work admits a seam, create the
gate and drive it red-green-refactor with aw-implement's own `aw-tdd`, and the signal is every
such gate green; only where no runnable gate exists at all — not merely where you'd skip one —
does your own completion report stand in, enumerating the brief's behaviours and asserting each
is covered; a weaker signal never substitutes where a gate was possible. What
bounds the loop depends on the run: an isolated run converges through
`skills/_shared/execution-convention.md`, held to an iteration ceiling (the dispatcher's, else
the contract's conservative default) whose fail-fast exit is never a pass; an attended build is
bounded by the human driving it. A run that stops short of its done-signal reports the shortfall
and ends here — no review, no landing (an isolated run's branch stays as its ref). Done when the
done-signal has fired.

### 4. Review the run

Only a run whose done-signal fired reaches here — a fail-fast run never does. Hand the change to
`aw-review-work` as an explicit review request — it only reviews and reports; you address what it
surfaces, and a behavioural fix re-enters `converge` (through `aw-tdd`), still ceiling-bounded, so
it lands back under the same done-signal. Done when the review passes or its findings are
addressed.

### 5. Land it — only when asked

Where the converged work lands depends on where it ran. An attended run on the current branch —
commit it there with a conventional-commit message, no attribution, but only when the user has
asked; that commit is a local landing, not a fold. Otherwise stop and report, ready to commit. A
run that went through `isolate` sits on its own named branch: always leave that branch as a ref,
since folding it into the mainline — the serial merge and whole-tree gate — is beyond this run,
as is resolving the fetched ticket; who folds it, or whether a fold-owner exists yet, is not
this run's to know. Done when an attended run is committed or reported ready, or an isolated
run's branch is left as a ref.

## Attribution

Adapted for this workspace from **Matt Pocock's `implement`** (github.com/mattpocock/skills,
Apache-2.0). Changes: switched to model-invocation and added a **two-gate guard** (magnitude
+ ripeness, whose definitions `aw-write-spec` owns) so the harness itself decides whether to
build or route back to planning; `/tdd` and `/code-review` become the harness's `aw-tdd` and
`aw-review-work` (which reviews only); work is `fetch`ed through the artifact-convention, and
an isolated run is `isolate`d and `converge`d through the execution-convention — the run-of-one
substrate this skill consumes as a single run, blind to any siblings a future orchestrator
dispatches beside it. Matt commits straight to the current branch; here that is gated on the
user having asked, and an isolated run's fold into the mainline is left to its fold-owner. The
spec-or-tickets input and test-first-at-seams spine are preserved.
