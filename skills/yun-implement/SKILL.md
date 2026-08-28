---
name: yun-implement
description: Build the work — write the code for a feature or fix (from a spec, tickets, or a described task), test-first at the agreed seams, then review and land it when asked. Use when the next step is writing code, or the user says "implement" / "implementá" / "buildeá". Declines large or unplanned work, routing it back to planning.
---

# yun-implement

Build the work — where a plan becomes code. It runs two ways from
one skill: **standalone** on a small, clear task, or as the last step of the heavy flow
(`yun-grill-plan` → `yun-write-spec` → `yun-slice-plan` → here). The skill itself decides which
it is.

## Guard — two gates, from the build side

Read the work before writing a line. `yun-write-spec` gates the same task from the spec side
with two tests — **magnitude** (small vs large) and **ripeness** (settled vs open decision).
Apply those same two here, and let its guard own their definitions so the two never drift.

- **Small and settled — build now.** Go straight to the steps: no spec, no tickets, no
  ceremony.
- **Large, or any blocking decision still open — plan first.** Don't improvise it: route
  back — `yun-grill-plan` if a decision is open, `yun-write-spec` / `yun-slice-plan` if it's
  settled but unsliced. Building large or unripe work is the failure this gate exists to stop.
- **One run, one ticket.** yun-implement builds a single ticket or task. A multi-ticket plan is
  a frontier to walk, not a run: an unsliced one routes back to planning, and an already-sliced
  one belongs to `yun-build-plan`. That skill is user-invoked, so this one cannot reach it — tell
  the user to type it rather than selecting one ticket and inventorying the rest.

Done when the task is confirmed small and settled, handed back to the planning flow, or named as
a walk for the user to start.

## Dispatched runs

A **dispatched run** is this skill entered on a walk's hand-off — one ticket, its repo, its base,
the name to give the workspace and branch, and no human at the convergence points. **What makes it dispatched is that package, never the
mechanism that carried it**: a walk with a subagent capability hands it to a fresh context, and one
without hands it to itself, in-thread. Both are dispatched runs and owe everything below. Reading
the mode off the mechanism instead would turn a walk's degraded pass into an attended build —
re-litigating whether to build at all, stopping for a review with nobody there, and landing a commit
its caller never asked for.

It differs from a run a human is driving in exactly four ways, and nowhere else: the guard
above is already satisfied — the walk found the ticket fit, so do not re-litigate magnitude or
ripeness; step 2 does not choose, it always isolates, passing the handed base and name; **it ends at
step 3** — neither the review of step 4 nor the landing of step 5 runs, because `yun-review-work`
assumes a human at its convergence points and no human is there, so review is the walk's to push
right, over the assembled result; and before ending it runs **the repo's whole-tree gates** — test,
build and typecheck, from wherever the repo declares them — against its branch in the workspace it
already built, reporting which of *passed*, *failed*, *none declared* or *could not run* it got.

That last one is not the ticket's done-signal, which step 3 already settled on the ticket's own
seams. It is a different question — does everything still hold together — and it is asked here
because this workspace is the only place the project's environment stands up. The walk chains or
stops on that answer, so a run that omits it hands back a branch its caller cannot act on.

So a **converged** dispatched run's own last act is to hand back **the name of the branch it
created** — the walk addresses it rather than predicting it — and to **leave standing the workspace
it created around that branch**. The branch carries the work, but only the workspace carries the
environment that work runs in: tear it down and a human must rebuild that environment before they
can run anything, which is how a chain reaches review unverified. **A run that fail-fasts removes
nothing** either: its workspace is the only evidence of what went wrong. Either way the workspace is
**a human's to remove**, once they have run what it holds — so a run reports its path alongside its
branch. The walk cuts its next run from a converged branch rather than from the mainline; a branch
you isolated on your own waits as a ref for a human.

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
owns, not an integration. A ticket handed to you to run, or a large or risky change worth
isolating, calls `isolate` through the contract, which resolves the run's workspace down its ladder (a sandbox,
else an isolated worktree via `yun-spawn-worktree`, else the host tree's single-writer floor),
every tier on its own named branch. **Pass on the base and the name you were handed** — the base
is the ref carrying this ticket's dependencies, the name is how your caller will address the
branch afterwards; a run you entered on your own has neither, so the contract cuts from HEAD and
the run names itself. Done when the workspace is chosen: the current branch for a task you build
in place, or an isolated workspace on its own named branch, cut from the right base and carrying
the name your caller will look for.

### 3. Converge on done

Loop the build until it reaches its done-signal: wherever the work admits a seam, create the
gate and drive it red-green-refactor with yun-implement's own `yun-tdd`, and the signal is every
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

Only a run whose done-signal fired reaches here — a fail-fast run never does, and neither does a
dispatched run, which ended at step 3. Hand the change to
`yun-review-work` as an explicit review request — it only reviews and reports; you address what it
surfaces, and a behavioural fix re-enters `converge` (through `yun-tdd`), still ceiling-bounded, so
it lands back under the same done-signal. Done when the review passes or its findings are
addressed.

### 5. Land it — only when asked

Where the converged work lands depends on where it ran. An attended run on the current branch —
commit it there with a conventional-commit message, no attribution, but only when the user has
asked; that commit is a local landing, not an integration. Otherwise stop and report, ready to
commit. A run that went through `isolate` sits on its own named branch: always leave that branch as
a ref, since integrating it — the whole-tree gate, and whatever a dispatcher chains onto it — is
beyond this run, as is resolving the fetched ticket. Done when an attended run is committed or reported ready, or an
isolated run's branch is left as a ref.

## Attribution

Adapted for this workspace from **Matt Pocock's `implement`** (github.com/mattpocock/skills,
MIT). Changes: switched to model-invocation and added a **two-gate guard** (magnitude
+ ripeness, whose definitions `yun-write-spec` owns) so the harness itself decides whether to
build or route back to planning; `/tdd` and `/code-review` become the harness's `yun-tdd` and
`yun-review-work` (which reviews only); work is `fetch`ed through the artifact-convention, and
an isolated run is `isolate`d and `converge`d through the execution-convention — the run-of-one
substrate this skill consumes as a single run, blind to any siblings a future orchestrator
dispatches beside it. Matt commits straight to the current branch; here that is gated on the
user having asked, and an isolated run's branch is left for whoever integrates it. The
spec-or-tickets input and test-first-at-seams spine are preserved.
