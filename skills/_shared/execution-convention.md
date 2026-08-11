# Execution convention

Shared contract for every `aw-*` skill that **runs** a single ticket to completion — takes one
branch-producing ticket it is handed and drives it, isolated, until its work reaches the strongest
done-signal the ticket offers and lands on a branch. Where `flow-convention.md`
**walks** the whole map — which ticket is takeable, how many run at once, at what cadence — this
contract governs the **run of one**: the isolated workspace that one ticket executes in, and the
loop that drives it to done. A skill declares **intent** — `isolate` or `converge` — and this file
maps both to whatever runtime the running agent has.

## Rules

1. **A skill NEVER names a runtime product.** Not a sandbox product, not a container engine.
   Naming one couples the harness to a product; the harness is tool-agnostic by design. A skill
   says *"run this ticket to completion"* — nothing more. Pointing at another harness contract or
   at a harness skill — as this file points at `aw-spawn-worktree` — is internal wiring, not a
   product name, and is allowed, exactly as `flow-convention.md` names `artifact-convention.md`.
2. **The run of one — execution never counts.** A run drives exactly one ticket and has no view of
   its siblings. How many run, in what order, which are takeable — **the count** — is
   `flow-convention.md`'s, not execution's. (Whether a ticket may run unattended is a separate flow
   concern — autonomy, not the count.) Execution isolates its one ticket so it is safe *beside*
   others without ever knowing there are others.
3. **The isolation boundary is one repo.** A run's isolated workspace spans a single repo;
   cross-repo coordination is out of scope.

## Mechanism

`isolate` resolves to the first tier the agent has, in order. `converge` is a loop, not a ladder.
Both below:

```
isolate(ticket, project):  the workspace the run happens in + its life in git — ALWAYS a
     distinct named branch: the unit a later integration layer folds back (it folds branches, not
     working trees), so every tier down to the floor must hand one up. What the tiers vary is the
     DEGREE of isolation around that branch — and that degree, not the branch, is what makes a run
     safe beside others. Resolve to the first available:
  1. A sandbox capability — an isolated container or VM the agent can drive → run the ticket on
         its own named branch inside its own sandbox. Isolation of both filesystem and git.
  2. Else → an isolated git worktree on its own named branch under .worktrees/<repo>/<task>/ ,
         via aw-spawn-worktree — filesystem isolation without a container, the lighter rung.
  3. Else → inline in the host working tree, on its own named branch. A host working tree holds
         one checked-out branch, so two inline runs in one repo cannot coexist: this floor is
         single-writer, and concurrent inline execution is unsupported — the same honest limit
         `flow-convention.md`'s file floor draws, reached here from execution's own lack of a
         second workspace, not by reading the walk's state. No isolation beside a sibling; safe
         only because it is the one writer.

converge(ticket):  loop the run — drive the work, evaluate the done-signal, and on anything short
     of it iterate again — with two exits only: the done-signal fires (success), or an iteration
     ceiling is reached first (a fail-fast surface, never a done). A failing gate or an unfinished
     run is not an exit — it spends one more iteration. The ceiling — a run-level bound the
     dispatching skill sets, and a conservative default when it does not — always bounds the loop,
     so it cannot spin forever. The done-signal is the strongest the ticket offers, and a weaker
     signal never substitutes for a stronger one the ticket carries:
  - The run's work has any runnable gate — a test, a build, an automated acceptance check → the
         signal is ALL such gates passing.
  - It has none → the signal is the agent's own completion report, the best available when there
         is nothing to run.
  A ticket held for a human never reaches this loop, so the completion-report tier is never a
  stand-in for the human. Whether the whole mainline still holds once this branch is folded in is
  a SEPARATE, serial concern (below) — not this loop's.
```

- **Sandbox capability** = any isolated runtime the agent can drive — a container, a microVM, a
  remote sandbox. It carries the ticket's whole run; the convention does not know or care which
  one it is.
- **A run produces its own named branch — never the mainline.** Every tier hands up one named
  branch, taken to the strongest done-signal the ticket offered. Folding that branch into the
  mainline — serially, resolving conflicts, and running the whole-tree gate that asks whether
  *everything still holds together* — is a distinct concern **beyond this contract**, one it hands
  its branch to and never performs: it must hold a whole-set view of every branch at once, which
  execution, seeing only its one, never can. This contract does not name who owns that fold or when
  it exists: today, folding one branch may be a trivial fast-forward with no whole-tree gate at
  all; a dedicated integration layer — a later convention or skill — may own the serial merge
  tomorrow. Either way, execution's run ends with that named branch left as a ref the fold-owner
  can address; folding it is not execution's.

## Scope resolution

`project` and the `ticket` resolve exactly as in `artifact-convention.md` — execution fetches the
ticket it is handed from that store. A dispatching skill hands it over, having already found it fit
to run: takeable as `flow-convention.md` defines it, unattended as its autonomy rule judges, and
branch-producing.
Execution does not select the ticket, compute a frontier, judge its autonomy, or decide it produces
a branch — composing those judgments into a hand-off is the dispatching skill's, one rung up.
Execution receives a ticket already found fit, and runs it.

## Ownership

Execution owns **isolation and convergence** — where one ticket runs, and that it reaches the
strongest done-signal the ticket offers on its own terms. Its whole deliverable is that one named
branch; closing or resolving the ticket is not execution's. It does NOT own: which ticket runs, how
many, or in what order (the count — flow's); whether the ticket may run unattended (flow's
autonomy rule, which the walk applies); which agent or model drives the work (the dispatching
skill's); or folding the finished branch back (the serial concern beyond this contract, handed the
branch but not owned here). Work that produces no branch — a finding, a decision — needs no
isolation and does not run here. The ticket's *resolution content* — the code or artifact the run produces — is
owned by the skill its type dispatches to, not by this contract.
