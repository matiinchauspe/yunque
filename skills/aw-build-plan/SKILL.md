---
name: aw-build-plan
description: Build an already-sliced plan to completion, leaving one branch to review.
disable-model-invocation: true
---

# aw-build-plan

Take a plan already sliced into **tracer bullets** and build it to completion, leaving ONE branch
for a human to review and merge. The **walk**: read the chain and compute the frontier → dispatch
one ticket to a run → resolve the ticket → read again, until the frontier drains or the walk stops
on something it must surface.

**The walk never writes to git and keeps no state of its own.** Each run is cut from the branch the
last one produced, so the chain integrates *by construction*: the newest branch already carries
every ticket before it, and the last one **is** the deliverable. Every pass reads that chain back
out of git and the ticket store rather than remembering it, which makes **every pass a resume** —
the first and the seventh run the same steps.

[`LIMITS.md`](LIMITS.md) holds one shape this walk refuses to build, one judgment it cannot make and
proceeds anyway, and every place the mechanism is weaker than it looks — including why a green gate,
a chained branch, and a resolved ticket each mean less than they appear to.

## Inputs

A human types this skill, plausibly bare. Ask for whichever of the first three you were not given:

- `<repo>` — the synced repo under `repos/<repo>/` that every ticket builds in.
- `<feature key>` — the key scoping the ticket set, as `artifact-convention.md` resolves it.
- `<slug>` — kebab-case, prefixing every run's workspace and branch so the whole chain shares one
  `aw/<slug>/…` namespace. Not decoration: it is **what step 1 reads the chain back out of**. Derive
  it from the feature key unless the user names one — but where tickets already read `resolved`,
  **ask for the slug that chain was built under**, or a derived one opens a second namespace beside
  it and the chain becomes unreadable.
- `<base>` — the ref the chain starts from, where that is not the mainline: a feature branch a fresh
  plan builds onto, or the chain a broken derivation cannot read for itself. **Where step 1 can
  derive a tip, that tip wins** — a base supplied over it would override a fact with a guess.

## Preconditions

- **The work is already sliced.** Implementation tickets published with their blocking edges drawn.
  If it isn't, hand back to `aw-slice-plan` — there is nothing here to walk.
- **`repos/<repo>/` is synced and a full clone.** `aw-spawn-worktree` requires both and answers
  either one missing by stopping to ask a human who is not there. Hand back to `aw-sync-repo`,
  rather than discovering it one failed dispatch at a time.
- **One walk per feature — and at the contract's bottom tier, one walk per repo.** It holds no lock.
  Two walks on one feature build two divergent chains, and since neither contains the other, no
  branch is the deliverable. Lower down the collision is coarser: the inline floor
  `execution-convention` degrades to is single-writer **per repo**, whatever feature it builds.

## Steps

### 1. Read the chain, then compute the frontier

Two reads, from scratch, **every pass** — nothing below is carried from the last one:

- `list` the **implementation** namespace through `skills/_shared/artifact-convention.md` for every
  ticket's number, blocking edges and status. What that set means for a walk is this walk's to say;
  the contract hands over tickets, never a frontier.
- `git -C repos/<repo> branch --list 'aw/<slug>/*'` — never a bare `git`, never a `cd`. Every branch
  this walk ever left stands here, one per dispatch, under the ticket number it ran for.

Cross them, and two facts fall out:

- **The chain's tip.** **No ticket reads `resolved`** → nothing is chained here, whatever failed
  branches are standing, and the base is the `<base>` you were given — confirm it resolves in the
  repo before dispatching onto it — else the mainline:
  `origin/HEAD` where it resolves, else whatever HEAD has checked out — record which, since a stale
  `origin/HEAD` is the human's to correct and this walk never fetches. **Otherwise every resolved
  ticket must have a branch** under its number, and one of those branches must hold all the rest as
  ancestors: that one is the tip, the deliverable so far, and the next dispatch's base. Ancestry is
  what orders them, since dispatch follows the edges rather than the numbers. **Check it per ticket,
  because the aggregate lies** — prune one branch and the newest survivor still holds every
  *remaining* branch as an ancestor, so it reads as a clean tip while that ticket's work sits outside
  the deliverable.
- **The failed set** — every ticket reading `open` that has a branch under its number. It was
  dispatched and never resolved. **Skip it, permanently.**

The frontier is what remains: **open**, **unblocked** — every edge `list` returns for it already
`resolved` — and **not in the failed set**, in number order, minus whatever step 2 declined on this
pass. Only `resolved` unblocks: a blocker closed any other way left no code behind, so a ticket that
declared a dependency on it would build against a hole.

**Deriving is the point, not an optimization.** A skip set kept in this thread dies with a
compaction, and the frontier then re-offers the failed ticket forever: dispatch, fail, skip, forget,
dispatch. State in context is tolerable **only when losing it costs a re-read rather than a
different answer** — true of step 2's declines, false of everything above. Derived, the bound
survives a restart too: every pass resolves a ticket, leaves a branch on one, or stops, all three
monotonic over a finite set **recorded in the repo**.

**Permanent is a decision.** A failed run's branch is the record that it ran, so **deleting that
branch is how a human retries the ticket.** The walk never writes to git, which makes a human's git
commands the entire control surface — and nothing needs syncing, because the repo *is* the state.

**When the derivation breaks** — a resolved ticket without its branch, or no branch holding the rest
as ancestors — neither the chain nor a base is readable.
**Stop and ask for `<base>`**, or take the one you were given, and confirm it resolves in the repo
before going on. Two rules here, not one: **the walk never picks the mainline itself**, because that
would look like a fresh walk and drop every resolved ticket out of the deliverable in silence — but
a human **may** name it, and once a human has merged the deliverable into the mainline and pruned
the refs, the mainline is the correct answer. Either way nothing verifies that base, so step 4
carries it as an assumption and says what rides on it.

**An empty frontier has three readings, and only one is a finished walk. Check them in this order,
and go to step 4 with whichever one holds:**

- **No tickets at all under this feature key** → nothing was ever sliced here. Say that, rather than
  reporting a finished walk over an empty set.
- **Nothing open remains** → the plan is built.
- **Open tickets remain** → separate them. Ones in the failed set are reported as skipped; the rest
  are **stranded**, each waiting — directly, or through a chain of open tickets — on something that
  will never resolve: a skipped ticket, a cycle among the edges, a number never published, or a
  blocker closed some way that is not `resolved`. Follow each one's edges to that root and name it.
  A walk that built eleven of twelve and skipped one is neither finished nor simply stuck.

Done when you hold the takeable set and the base, or know which empty reading you are in.

### 2. Dispatch the frontier's first ticket to a run

**Read the host working tree first — `git -C repos/<repo> status --porcelain` — and stop the walk
when anything is loose there.** A run that fail-fasts at the contract's bottom tier leaves its edits
in that tree, and the next run commits them under **its own** ticket, past ancestry, commits and a
green gate alike. Nothing tells you in advance which tier a run resolves to, so the read is
unconditional — and it is a read rather than a report, because a run that dies reports nothing while
the tree still shows it.

`fetch` the ticket through `artifact-convention.md` — a second read of a ticket `list` already
returned, deliberately: the walk needs the body, the subagent needs its own copy in a fresh context.
**An unreachable ticket declines that fetch, not this walk** — record the decline and return to step
1, rather than burying a plan behind the one ticket nobody can read.

A body carrying an **`Integrates on:`** line **that names a branch** is an expand–contract batch and
is **declined**. [`LIMITS.md`](LIMITS.md) holds why, and why only a named branch counts.

Hand the ordinary ticket to a **subagent**, whose fresh context shares nothing with this thread.
Tell it explicitly that this is a **dispatched run** of `aw-implement`, and give it the repo, the
**feature key and the implementation namespace** it needs to `fetch` the ticket for itself, the
ticket's number, the **base** step 1 derived, and the **`name`** `aw/<slug>/<NN>-<random>` — this
walk's slug, the ticket's number, and a random suffix, which is what the source template settled on
after second-granularity timestamps collided. Step 1 matches on the `<NN>-` prefix, so the suffix
costs the walk nothing.

`aw-implement`'s dispatched-run rules then apply, and one of them is this walk's whole review
posture: **it stops before review and before landing**, because review is pushed right, to the
human, over the assembled result. Let `execution-convention`'s own default bound the run's
iterations.

Without a subagent capability, hand that same package to yourself. It is **still a dispatched run** —
`aw-implement` keys the mode off the package, never off who carries it — so only the fresh context
is lost, and step 4 says so.

Done when the ticket is dispatched, or a decline is recorded and step 1 recomputed.

### 3. Take what the run hands back

**The run's verdict decides the move; the branch can only veto it.** Read them in that order. A run
that fail-fasted leaves its branch standing with partial commits on it — `aw-implement` step 3 ends
there and leaves the ref — so a branch that is present, descended and non-empty proves something was
written, never that the ticket reached its done-signal.

Three moves end a pass. **Chain**: the ticket resolves, its branch is the next base, step 1 reads
again. **Skip**: the chain does not advance, the next dispatch cuts from the same base, and you
**leave everything the run left behind** — that branch is what step 1 reads to know this ticket was
tried, and its workspace is a human's to inspect. Only a *converged* run owes you its branch name,
so for a skip, report the name you handed out. **Stop the walk**: leave every branch where it is,
and go to step 4 with what stopped it.

**The whole-tree verdict picks the move** — a question answered inside the run, in the only workspace
where the project's environment stands up:

- **Nothing came back** — no branch named, or none that resolves in the repo → **stop the walk**,
  naming the ticket. Every tier owes one named branch, so nothing at all means the isolation layer
  failed rather than the ticket, and it fails the same way for every ticket behind it. It is also the
  one outcome step 1 cannot record: an open ticket with no branch reads exactly like one never
  dispatched, so carrying on re-offers it forever.
- **Could not run the gates** → **skip**, reporting a broken gate. It says nothing about the ticket,
  so it is never a slicing finding.
- **Failed** → **stop the walk.** A red whole-tree gate is the one verdict that can be evidence
  about work **already chained**: the defect may be this ticket's, or it may have been sitting in
  the chain for several tickets with only this one's gates sharp enough to expose it. Skipping would
  cut the next run from that same chain, whose gates may not cover the defect at all — so the walk
  stops and lets a human read which it is. Step 4 must not pick a reading, and a red on the
  **first** ticket chained points at the base rather than at the slicing.
- **Passed**, or **the repo declares none** → the branch may carry the chain, subject to the two
  vetoes. No gates is not a pass — step 4 reports it unverified.

**Then the two vetoes, neither of which asks the run:**

- **Ancestry** — is the base you handed out an ancestor of the branch? `execution-convention:114-123`
  requires this of any caller that supplied one, and spells out what a lost base returns and why
  nothing downstream catches it. Not an ancestor, or the ref unreadable → **stop the walk**, naming
  the ticket whose branch broke it.
- **Commits** — does anything sit between that base and the branch? A branch level with its base
  built nothing, and resolving its ticket would record work that is not there. Nothing → **skip**.

**Compare the `name` you gave against the one the run reports.** A tier that could not honour it says
which it used, and a branch outside `aw/<slug>/<NN>-` is invisible to step 1 — the next pass reads
this ticket as open with no branch and **redispatches work already built**. Carry on with the walk,
and let step 4 declare this chain no longer derivable, naming every branch: from here it resumes
only on a `<base>` a human supplies.

Chaining `resolve`s the ticket through `artifact-convention.md`. **Resolved means integrated, not
delivered** — the deliverable still waits for a human, and `resolve` has no inverse.

**If `resolve` fails, stop the walk.** It leaves exactly the signature a failed run leaves — an open
ticket with a branch under its number, descended from the tip — and nothing in git separates the
two, not ancestry, not commits, not order. A later pass would read this built, gated, chained work as
a failed attempt and skip it forever, while the tip fell back to the ticket before it. **The report
is the only carrier**: step 4 owes it the branch, the number, and the two things a human can do that
mean anything here — `resolve` the ticket by hand, so the next pass reads the chain whole, or delete
the branch, so the ticket is retried from scratch. **Restarting without doing either discards that
run's work in silence.**

Done when the pass has chained, skipped, or stopped.

### 4. Report

- **Why the walk ended** — drained, drained with skips, stuck, nothing sliced, or stopped, and on
  what.
- **The deliverable — the chain's tip**: the branch the last chained run **produced**, named as the
  one to review and merge. Where nothing chained, a resumed walk's deliverable is the base it started
  from, unchanged; only a *fresh* walk that chained nothing has **no deliverable**. Where a failed
  `resolve` stopped the walk, the tip still is the deliverable and its newest ticket reads open — say
  both, or a later walk re-offers work that tip already holds.
- **What it was cut from** — the derived tip, the mainline snapshot step 1 recorded, or a `<base>` a
  human named. Say plainly when it was the last: on a feature with resolved tickets, nothing checked
  that base carries their work, and if it does not, that work is not in the deliverable.
- **The chain** — every ticket built, in order, with the branch each produced.
- **Every ticket skipped, loudly**, with which of step 3's reasons skipped it, naming any branch and
  workspace it left behind and saying that **deleting the branch is what retries it**. For a
  declined batch, what it would take to build by hand.
- **Every ticket left stranded**, each with the root its edges lead to. A quiet strand reads as a
  delivery.
- **What was never verified** — every ticket whose repo declared no gates, and any run that shared
  this thread's context rather than a subagent's.
- **Whether this chain is still derivable**, wherever step 1 could not read a tip or step 3 found a
  name outside the namespace.

Then say **when the refs may go: once the whole feature is built — not once a human has merged the
deliverable into the mainline.** Step 1 reads the chain out of that namespace, and a merged
deliverable over a feature with tickets still open is precisely the walk that will need it.

Done when the human knows what to review, what was skipped, what was stranded, what went unchecked,
and what a retry costs.

## Attribution

Adapted for this workspace from **Matt Pocock's `parallel-planner`** template
(github.com/mattpocock/sandcastle, MIT). Preserved: the **frontier** of unblocked tickets, the loop
that recomputes it as work lands, and the rule that a run which committed nothing has built nothing.

Changed: **serial rather than fanned out**; the recompute is per ticket where the source's is per
round; the whole-tree gate runs inside each ticket's own run where the source runs it after each
merge; a red gate stops the walk instead of being fixed forward, since the harness runs against
repos it does not own; and the walk holds no state, deriving each pass from git and the ticket store
where the source's planner carries its own.

**Serial rests on the predicate, not on the floors.** The source's frontier treats file overlap as a
blocking edge; ours are the dependency edges the slicing declared, so two of our frontier's tickets
may well touch the same files — fanning out would generate exactly the conflicts the source's merge
phase exists to resolve, minus the planning-time disjointness that keeps that phase tractable. And
`aw-slice-plan:33` makes every slice cut "a narrow but complete path through every layer", so that
overlap is structural: put the term back and the frontier collapses to roughly one ticket. Second,
`isolate` resolves its ladder *inside* the run, so the walk **cannot know before dispatching** which
tier a run lands on, and must plan for the single-writer floor.
`docs/research/matt-pocock-sandcastle-parallelism.md` holds the evidence.
