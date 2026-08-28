# Execution convention

Shared contract for every `yun-*` skill that **runs** a single ticket to completion — takes one
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
   at a harness skill — as this file points at `yun-spawn-worktree` — is internal wiring, not a
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
isolate(ticket, project, base?, name?):  the workspace the run happens in + its life in git — ALWAYS a
     distinct named branch, cut from `base`: the unit a later integration layer takes (it takes
     branches, not working trees), so every tier down to the floor must hand one up. What
     the tiers vary is the DEGREE of isolation around that branch — and that degree, not the
     branch, is what makes a run safe beside others. **`base` is optional, and a caller holding
     one MUST pass it**: it is the ref already carrying every dependency this ticket declares,
     and only a caller with the whole-set view knows which ref that is — a run, seeing one
     ticket, cannot compute it. Absent a base, the run cuts from the repo's current HEAD and says
     so in its report: right for a standalone run, wrong for a dispatched one, which is why the
     dispatcher always supplies it. **`name` is optional, and a caller that will need to address
     this branch again MUST pass it**: it names both the workspace and the branch the run leaves,
     so a dispatcher can group a whole set under one namespace — `<prefix>/<ticket>-<nonce>`, the
     nonce because a second attempt at one ticket must not collide with what the first left
     behind — and later list, prune, or re-derive that set from git alone. Absent a name the run
     invents one and reports it: right for a run nobody will come back to, wrong for a set someone
     must reassemble, which is why a dispatcher holding a namespace supplies it. A tier that
     cannot honour the name it was given reports the name it actually used, exactly as it reports
     a base it could not take. Resolve to the first available:
  1. A sandbox capability — an isolated container or VM, either one the agent has loaded or one
         the TARGET REPO declares to agents beside its own instructions. A repo that brings its
         own wins, since it knows how the project builds. Whatever its source, it qualifies at
         this rung only by SHAPE: a PER-TICKET entry point that, invoked with the ticket, the
         base, and the name where it takes one, runs that one ticket isolated and leaves one
         named branch. A whole orchestrator —
         a runner that plans, fans out and merges by itself — is NOT this rung: it answers a
         different verb, and taking it here would trade the caller's walk away. What it adds
         over rung 2 is **filesystem** isolation — a container wrapped around the same branch,
         not a separate git universe — and **the branch it leaves is always a ref in the host
         repo**: a runtime that mounts the host's own workspace gets that for free, one that
         runs the work elsewhere must sync the ref back, and one that can do neither has no
         tier-1 capability here, so the ladder degrades. Every rung hands up the same artifact;
         only the degree of isolation around it varies.
  2. Else → an isolated git worktree on its own named branch, cut from `base`, under
         .worktrees/<repo>/<task>/ , via yun-spawn-worktree — where `name` is that skill's
         `<task>`, which names the worktree dir and the branch alike. Filesystem isolation
         without a container, the lighter rung.
  3. Else → inline in the host working tree, on its own named branch cut from `base` and
         called `name`. A host
         working tree holds one checked-out branch, so two inline runs in one repo cannot
         coexist: this floor is
         single-writer, and concurrent inline execution is unsupported — the same honest limit
         `flow-convention.md`'s file floor draws, reached here from execution's own lack of a
         second workspace, not by reading the walk's state. No isolation beside a sibling; safe
         only because it is the one writer. **Sequential runs share this tree too, so an inline
         run MUST leave it clean** — everything committed to its branch, or reported and nothing
         left loose. A run that cannot, because it fail-fasted mid-build, leaves the floor
         UNUSABLE: git carries loose changes across the next checkout, so the following run would
         commit the failed one's edits under its own ticket and gate them green. Such a run says
         the floor is dirty, and a caller sequencing runs stops there rather than dispatch onto
         it — the tier that is always available is also the one that cannot isolate a failure.

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
  stand-in for the human. Whether the whole tree still holds once this branch is integrated is
  a SEPARATE, serial concern (below) — not this loop's.
```

- **Sandbox capability** = any isolated runtime the agent can drive — a container, a microVM, a
  remote sandbox. It carries the ticket's whole run; the convention does not know or care which
  one it is, only that it takes one ticket at a time. A runtime that offers no per-ticket entry
  point simply has no tier-1 capability here — the ladder degrades to the worktree, which is why
  every rung answers the same verb: one ticket in, one named branch out.
- **A run produces its own named branch — never the mainline.** Every tier hands up one named
  branch, taken to the strongest done-signal the ticket offered. **Integrating** that branch —
  serially, behind the whole-tree gate that asks whether *everything still holds together* — is a
  distinct concern **beyond this contract**, one it hands its branch to and never performs: it must
  hold a whole-set view of every branch at once, which execution, seeing only its one, never can.
  The dispatching skill owns it, holding that view and having handed this run its base. A **serial**
  dispatcher owns it cheaply: it integrates by handing this branch on as the NEXT run's `base`, so
  the branches never diverge and integration costs no git write at all. Where no such caller is
  present, nobody integrates: the branch simply waits as a ref for a human. Either way, execution's
  run ends with that named branch left as a ref its integrator can address.
- **A caller that supplied a `base` MUST confirm the branch it gets back descends from it.** The
  run reports the base it actually cut from — that is what the HEAD fallback above says out loud —
  and a run can lose the base without failing: a relay that drops it, a tier-1 entry point that
  takes none, a runtime that seeds from the repo's default branch. What comes back then is a branch
  that is green, plausible, and missing everything the base carried. **Nothing downstream catches
  it**, because the work it dropped is already recorded as done, and a serial caller will cut every
  later run from it. Reading the base the run reports is the cheap check; where the caller can
  address the repo, asking git whether the base is an ancestor of the returned branch is the
  certain one, and it is a read. A caller that does neither has handed one run its whole
  integration, and owes that trust a declared limit rather than silence.

## Scope resolution

`project` and the `ticket` resolve exactly as in `artifact-convention.md` — execution fetches the
ticket it is handed from that store. **A run is dispatched when it was handed that package — one
ticket, its repo, its `base`, and no human at its convergence points — and never by the mechanism
that carried it.** A dispatcher with a fresh-context capability hands the package to one; a
dispatcher without hands it to itself and runs in its own thread. Both are dispatched, and the
skill the ticket's type resolves to owes each the same obligations and the same stopping point.
Keying the mode off the mechanism instead makes a degraded dispatcher silently become an attended
build — which is a mode change wearing the clothes of a context cost. A dispatching skill hands it
over with its `base`, having
already found it fit to run by its own judgment: takeable, branch-producing, and unattended where
its namespace carries an axis to judge that on — the implementation one does not, and the walk over
it says so rather than claiming the judgment. A
run entered without a dispatcher — a change a human chose to isolate — arrives with no base and
no such judgment, and takes the HEAD default above.
Execution does not select the ticket, compute a frontier, judge its autonomy, or decide it produces
a branch — composing those judgments into a hand-off is the dispatching skill's, one rung up.
Execution receives a ticket already found fit, and runs it.

## Ownership

Execution owns **isolation and convergence** — where one ticket runs, and that it reaches the
strongest done-signal the ticket offers on its own terms. Its whole deliverable is that one named
branch; closing or resolving the ticket is not execution's. It does NOT own: which ticket runs, how
many, or in what order (the count — flow's); whether the ticket may run unattended (flow's
autonomy rule — applied by a walk whose namespace carries an axis to judge it on, and declared
unjudged by one whose namespace does not); which agent or model drives the work (the dispatching
skill's); or integrating the finished branch (the serial concern beyond this contract, handed the
branch but not owned here). Work that produces no branch — a finding, a decision — needs no
isolation and does not run here. The ticket's *resolution content* — the code or artifact the run produces — is
owned by the skill its type dispatches to, not by this contract.
