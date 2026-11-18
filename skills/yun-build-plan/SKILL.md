---
name: yun-build-plan
description: Build an already-sliced plan to completion, leaving one branch to review.
disable-model-invocation: true
---

# yun-build-plan

Take a plan already sliced into **tracer bullets** and build it to completion, leaving ONE branch
for a human to review and merge. The **walk**: read the chain and compute the frontier → dispatch
one ticket to a run → resolve the ticket → read again, until the frontier drains or the walk stops
on something it must surface.

**The walk keeps no state of its own, and writes to git only to commit a run a human approved** —
in an attended walk; unattended, every git command it runs is a read. Each run is cut from the
branch the last one produced, so the chain integrates *by construction*: the newest branch already carries
every ticket before it, and the last one **is** the deliverable. Every pass reads that chain back
out of git and the ticket store rather than remembering it, which makes **every pass a resume** —
the first and the seventh run the same steps.

[`LIMITS.md`](LIMITS.md) holds one shape this walk refuses to build, one judgment it cannot make and
proceeds anyway, why the walk is serial rather than fanned out, and every place the mechanism is
weaker than it looks — including why a green gate, a chained branch, and a resolved ticket each mean
less than they appear to.

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
- **attended** — whether a human reviews each run's changes **uncommitted** before it chains. Ask
  when you were not told. Attended, the run leaves its work uncommitted and the walk commits it on
  the human's ok; unattended, the run commits and nothing waits on a person.

## Preconditions

- **The work is already sliced.** Implementation tickets published with their blocking edges drawn.
  If it isn't, hand back to `yun-slice-plan` — there is nothing here to walk.
- **`repos/<repo>/` is synced and a full clone.** `yun-spawn-worktree` requires both and answers
  either one missing by stopping to ask a human who is not there. Hand back to `yun-sync-repo`,
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
  repo before dispatching onto it — else the mainline, resolved by `execution-convention`'s rule and
  recorded as the ref it resolved to; a stale ref is the human's to correct, since this walk never
  fetches. **Otherwise every resolved
  ticket must have a branch** under its number, and one of those branches must hold all the rest as
  ancestors: that one is the tip, the deliverable so far, and the next dispatch's base. Ancestry is
  what orders them, since dispatch follows the edges rather than the numbers. **Check it per ticket,
  because the aggregate lies** — prune one branch and the newest survivor still holds every
  *remaining* branch as an ancestor, so it reads as a clean tip while that ticket's work sits outside
  the deliverable.
- **The failed set** — every ticket reading `open` that has a branch under its number. It was
  dispatched and never resolved. **Skip it, permanently.** Its newest log entry, read through
  `fetch`, overrides that four times. A **`parked`** entry → the ticket is **parked**: off the
  frontier, and reported with its question, never as skipped, whether or not its workspace still
  stands. An **`answered`** entry → the ticket is **answered**: takeable, and step 2 dispatches it
  as a resume. A **`checkpoint`** entry → takeable the same way, resumed from its baton. An
  *awaiting review* entry, with the workspace still standing with uncommitted changes → take it
  straight to step 3's stop for the human rather than dispatching, with the tip as the base it was
  handed, since nothing chained past it. An *awaiting review* ticket whose workspace is gone is
  failed like any other.

The frontier is what remains: **open**, **unblocked** — every edge `list` returns for it already
`resolved` — and **neither failed nor parked**, in number order, minus whatever step 2 declined on
this pass. Only `resolved` unblocks: a blocker closed any other way left no code behind, so a
ticket that declared a dependency on it would build against a hole.

**Deriving is the point, not an optimization.** A skip set kept in this thread dies with a
compaction, and the frontier then re-offers the failed ticket forever: dispatch, fail, skip, forget,
dispatch. State in context is tolerable **only when losing it costs a re-read rather than a
different answer** — true of step 2's declines, false of everything above. Derived, the bound
survives a restart too: every pass resolves a ticket, leaves a branch on one, or stops, all three
monotonic over a finite set **recorded in the repo** — and a resume spends its answer or baton in
a `resumed` entry before it leaves its branch, so each re-offers its ticket once, and step 3's
ceiling bounds how many batons one ticket writes.

**Permanent is a decision.** A failed run's branch is the record that it ran, so **deleting that
branch is how a human retries the ticket.** A ticket that took an answer or a baton has several:
append another `answered` entry to resume it from the one holding the others, or delete them all
and append one to start it fresh — the log outlives the branches, and only an `answered` entry
restarts step 3's ceiling. Past the commits a human approves, the walk never writes to git, which
makes a human's git commands the entire control surface — and nothing needs syncing, because the
repo *is* the state.

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
- **Open tickets remain** → separate them. Ones in the failed set are reported as skipped; parked
  ones as parked, and every ticket waiting on one — directly, or through a chain of open tickets —
  as **waiting on your answer in #NN**, naming the parked ticket at the chain's end, unless another
  of its edges leads to a root below, which strands it whatever the answer; ones step 2
  **declined** — a ticket nothing could `fetch`, or a batch refused — are takeable and wait on no
  root, so they are reported as declined rather than stranded; the rest are **stranded**, each
  waiting — directly, or through a chain of open tickets — on something that will never resolve: a
  skipped ticket, a cycle among the edges, a number never published, or a blocker closed some way
  that is not `resolved`. Follow each **stranded** one's edges to that root and name it.
  A walk that built eleven of twelve and skipped one is neither finished nor simply stuck.

Done when you hold the takeable set and the base, hold a run awaiting review, know which empty
reading you are in, or stopped because neither the chain nor a `<base>` resolved.

### 2. Dispatch the frontier's first ticket to a run

**Read the host working tree first — `git -C repos/<repo> status --porcelain` — and stop the walk
when anything is loose there.** A run that fail-fasts at the contract's bottom tier leaves its edits
in that tree, and the next run commits them under **its own** ticket, past ancestry, commits and a
green gate alike. Nothing tells you in advance which tier a run resolves to, so the read is
unconditional — and it is a read rather than a report, because a run that dies reports nothing while
the tree still shows it. **The one exception is a path this ticket's body names** — fetched just
below — since its run commits that path under the ticket that carries it: let it through, and step 4
reports it.

`fetch` the ticket through `artifact-convention.md` — a second read of a ticket `list` already
returned, deliberately: the walk needs the body, the subagent needs its own copy in a fresh context.
**An unreachable ticket declines that fetch, not this walk** — record the decline and return to step
1, rather than burying a plan behind the one ticket nobody can read.

A body carrying an **`Integrates on:`** line **that names a branch** is an expand–contract batch and
is **declined**. [`LIMITS.md`](LIMITS.md) holds why, and why only a named branch counts.

Hand the ordinary ticket to a **subagent**, whose fresh context shares nothing with this thread.
Tell it explicitly that this is a **dispatched run** of `yun-implement`, and give it the repo, the
**feature key and the implementation namespace** it needs to `fetch` the ticket for itself, the
ticket's number, the **base** step 1 derived, and the **`name`** `aw/<slug>/<NN>-<random>` — this
walk's slug, the ticket's number, and a random suffix, which is what the source template settled on
after second-granularity timestamps collided. Step 1 matches on the `<NN>-` prefix, so the suffix
costs the walk nothing. Quote `practice-convention`'s rule 2 paragraph **A comment carries the why**
into the package whole — a run left to reach it through conform mirrors the comments around it —
and ask the run to hand back step 3's comment table over its own diff. In an attended walk, also
tell it to leave its work **uncommitted** on its branch, in its workspace: the human reviews it there.

An **answered** or checkpointed ticket is a **resume**: the same package under a fresh `name`,
plus as `prior` the branch under that number holding its others — where several stand and none
holds the others, decline it, since a pick would leave one outside the chain; where none stands, it
goes without a `prior`. **A ticket past step 3's
ceiling is not dispatched**: append the `parked` entry step 3 owes it and return to step 1. Quote
into the package every `answered` entry, oldest first, binding over the framing, and the newest
`checkpoint` entry where no `resumed` entry follows it, the baton the run starts from — each where
the log holds one, since a run in a cloned workspace cannot read the log — and, in an attended
walk, say that the merge is committed and only the ticket's new work stays uncommitted.
Where the answer moves criteria to a new ticket, and no ticket blocked by this one carries them
yet, `publish` them first through `artifact-convention.md` as an implementation ticket blocked by
this one. **Before dispatching, append a `resumed` entry** naming the
`name` and `prior`, or `none`: the answer or baton is spent there, so short of chaining, parking or
checkpointing again the ticket reads failed like any other instead of re-offered each pass.

`yun-implement`'s dispatched-run rules then apply, and one of them is this walk's whole review
posture: **it stops before review and before landing**, because review is pushed right, to the
human, over the assembled result. Let `execution-convention`'s own default bound the run's
iterations.

Without a subagent capability, hand that same package to yourself. It is **still a dispatched run** —
`yun-implement` keys the mode off the package, never off who carries it — so only the fresh context
is lost, and step 4 says so.

Done when the ticket is dispatched, a decline is recorded and step 1 recomputed, or the walk stopped
on a loose host working tree.

### 3. Take what the run hands back

**The run's verdict decides the move; the branch can only veto it.** Read them in that order. A run
that fail-fasted leaves its branch standing with partial commits on it — `yun-implement` step 3 ends
there and leaves the ref — so a branch that is present, descended and non-empty proves something was
written, never that the ticket reached its done-signal.

Three moves end a pass. **Chain**: the ticket resolves, its branch is the next base, step 1 reads
again — and **the previously chained link's workspace is removed**, since the branch you just
chained already contains it and a human only ever needs to run the newest one. Prune **workspaces,
never refs**: `git worktree remove` deletes a working directory, and every branch this walk left
stands untouched, which is what keeps step 1 able to read the chain. A run cannot do this pruning
itself — it never knows whether it is the last — so the walk does it, one link behind. **Skip**: the
chain does not advance, the next dispatch cuts from the same base, and you
**leave everything the run left behind** — that branch is what step 1 reads to know this ticket was
tried, and its workspace is a human's to inspect. Only a *converged* run owes you its branch name,
so for a skip, report the name you handed out. **Stop the walk**: leave every branch where it is,
and go to step 4 with what stopped it.

**When a human sends a stopped run back to work**, hand it the **symptom** and the evidence — the
failing check, its output, the human's ruling — leave the fix to the run, and take what it hands back
through this same step. The run holds the
workspace and the evidence; the walk holds only its report, so a "don't touch X" the run has not
itself ruled out fences off exactly where the defect may sit.

**A run that parked, or one that reports done while its completion report leaves an acceptance
criterion unaccounted for, parks the ticket** — before anything below. Append a `parked` entry to
its log through `artifact-convention.md`: the run's question and recommendation, or the question
the gap raises and your recommendation, rather than dispatching it again. Then **skip**. In an
attended walk only the first parks, its work committed by the run; the gap goes to the stop for
the human below, who is at hand, and whose uncommitted work a later resume could not reach.

**A run that checkpointed** is handled before anything below but *nothing came back*. It closes its
report with a `## Handoff` section; append what sits below that heading, verbatim, as a `checkpoint`
entry, then **skip**: step 1 reads the entry as takeable and the next dispatch resumes it. **The
ceiling is 3**: a 4th or later `checkpoint` since the ticket's last `answered` entry, or since its
log began, also appends a `parked` entry. Its question is whether to keep going or which remaining
criteria move to a new ticket, and it lists the criteria done and remaining by the newest baton, how
many each baton closed, and every ticket blocked by this one. Recommend moving the remaining
criteria no blocked ticket relies on where a round closed none and some are movable, else keeping
going.

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

**Then the two vetoes, neither of which asks the run** — in an attended walk, only once the human's
ok has been committed, below:

- **Ancestry** — is the base you handed out an ancestor of the branch? `execution-convention:168-177`
  requires this of any caller that supplied one, and spells out what a lost base returns and why
  nothing downstream catches it. A resume asks the same of the `prior` its `resumed` entry names:
  a run that skipped that merge leaves the prior branch outside the chain, and step 1 then finds
  no tip. Not an ancestor, or the ref unreadable → **stop the walk**, naming the ticket whose branch
  broke it.
- **Commits** — does anything sit between that base and the branch? A branch level with its base
  built nothing, and resolving its ticket would record work that is not there. Nothing → **skip**.

**Compare the `name` you gave against the one the run reports.** A tier that could not honour it says
which it used, and a branch outside `aw/<slug>/<NN>-` is invisible to step 1 — the next pass reads
this ticket as open with no branch and **redispatches work already built**. Carry on with the walk,
and let step 4 declare this chain no longer derivable, naming every branch: from here it resumes
only on a `<base>` a human supplies.

**Before chaining, show the comments the run added** — every comment line its change adds over the
base you handed out: `git -C repos/<repo> diff <base> <branch>`, or, while an attended run's work is
uncommitted, `git -C <workspace> diff <base>` plus every untracked file it created, read whole — as
a table, one row each: the comment, what the code beside it already says, and *keep*, *cut* or a
one-line rewrite. Each row asks one question: delete it, and is anything lost the code does not say? A read with no table is not done —
the run's own table is a claim to check, not this read. Neither a veto nor a move: every row not
*keep* goes to step 4 by `file:line`, since a long diff hides it from the human.

**In an attended walk, stop for the human here**, before anything is committed and before the two
vetoes, which have no commits to read yet. Append an *awaiting review* entry to the ticket's log
through `artifact-convention.md` — the only record that this branch waits on a person, since in git
it reads as a failed run — and hand the human the workspace path, the files changed, your review
with the comment table, and the commits you would split the change into. **Their ruling picks the
move:** changes go back to the run by the rule above — or, where that run is gone with a past session,
the human chooses between the walk making them in that workspace, then this stop again, and
deleting the branch to retry the ticket; an ok has the walk commit on the run's branch,
in that workspace — conventional commits, no attribution — and only then run the two vetoes.

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

Done when the pass has chained, parked, skipped, stopped, or waits on the human's ruling.

### 4. Report

- **Why the walk ended** — drained, drained with skips or parked tickets, stuck, nothing sliced, or
  stopped, and on what.
- **The deliverable — the chain's tip**: the branch the last chained run **produced**, named as the
  one to review and merge. Where nothing chained, a resumed walk's deliverable is the base it started
  from, unchanged; only a *fresh* walk that chained nothing has **no deliverable**. Where a failed
  `resolve` stopped the walk, the tip still is the deliverable and its newest ticket reads open — say
  both, or a later walk re-offers work that tip already holds.
- **What it was cut from** — the derived tip, the mainline snapshot step 1 recorded, or a `<base>` a
  human named. Say plainly when it was the last: on a feature with resolved tickets, nothing checked
  that base carries their work, and if it does not, that work is not in the deliverable.
- **The chain** — every ticket built, in order, with the branch each produced, and **the one
  workspace still standing**: the deliverable's, which contains every link before it. That is where
  a human runs the gates before merging. Say plainly that the earlier workspaces were pruned as the
  walk chained past them and that their branches are all still there, or a reader will take the
  missing directories for lost work. Removing that last workspace is theirs to do, once they have
  run it, and so is removing any a parked or checkpointed run left, so name
  them.
- **The deliverable, running** — where its workspace is still standing, offer to start it there by
  whatever entry point the repo declares for running itself, and start it only on a yes. Where that
  workspace is gone or the repo declares no such entry point, say which rather than guess.
- **The deliverable, reviewed** — where there is a deliverable, offer `yun-review-work` over it,
  and run it only on a yes. Each fix it calls for is published as an implementation ticket blocked
  by the newest resolved one and walked from step 1, never edited inline: a branch with no ticket
  under its number is invisible to step 1.
- **Every ticket skipped, loudly**, with which of step 3's reasons skipped it, naming any branch and
  workspace it left behind and saying that **deleting the branch is what retries it** — or, for a
  ticket that took an answer or a baton, naming all its branches and step 1's two retries.
- **Every ticket declined**, with what step 2 refused it for: a batch, with what it would take to
  build by hand; a ticket nothing could `fetch`, with the store and key that failed — the repair is
  there rather than in the repo, and a declined ticket left no branch to delete; a resume whose
  branches none holds the others, naming them all, since deleting all but one makes it takeable.
- **Every ticket parked**, with its question, its recommendation, its branches, and their
  workspaces where they still stand — evidence only, since the resume runs fresh from the branch;
  then every ticket waiting on that answer, as **waiting on your answer in #NN** — apart from the stranded,
  since an answer frees them.
- **Every ticket left stranded**, each with the root its edges lead to. A quiet strand reads as a
  delivery.
- **Loose paths let through** — each one step 2 passed, with the ticket whose body named it.
- **Comments over the bar** — every row step 3's table did not keep, by ticket and `file:line`, with
  its rewrite, or "none" said outright.
- **What was never verified** — every ticket whose repo declared no gates, and any run that shared
  this thread's context rather than a subagent's.
- **Whether this chain is still derivable**, wherever step 1 could not read a tip or step 3 found a
  name outside the namespace.

Then say **when the refs may go: once the whole feature is built — not once a human has merged the
deliverable into the mainline.** Step 1 reads the chain out of that namespace, and a merged
deliverable over a feature with tickets still open is precisely the walk that will need it.

**Where the walk ended on an empty frontier with tickets parked, ask their questions** — never
after a stop, whose chain waits on a human's reading. A parked question waits for this point, so
nothing independent stalls behind it. Ask each one at a time, with its recommendation, and append
each answer as an `answered` entry through `artifact-convention.md`, written out as what it
settles — an accepted recommendation in full, never a bare yes. A question left unanswered stays
parked, and is not asked again this session. After the round, if any answer came, return to step 1
in this session. With none, the walk ends: relaunching the same
`/yun-build-plan <repo> <feature> <slug>` asks again, or takes an `answered` entry a human appended
to the log since.

Done when the human knows what to review, which comments to prune, which questions wait on them,
what was skipped, what was declined, what was stranded, what went unchecked, and what a retry
costs — has been offered the deliverable running and reviewed, or told why it cannot be — and,
after an empty frontier, has been asked every parked question.

## Attribution

Adapted for this workspace from **Matt Pocock's `parallel-planner`** template
(github.com/mattpocock/sandcastle, MIT). Preserved: the **frontier** of unblocked tickets, the loop
that recomputes it as work lands, and the rule that a run which committed nothing has built nothing.

Changed: **serial rather than fanned out**; the recompute is per ticket where the source's is per
round; the whole-tree gate runs inside each ticket's own run where the source runs it after each
merge; a red gate stops the walk instead of being fixed forward, since the harness runs against
repos it does not own; and the walk holds no state, deriving each pass from git and the ticket store
where the source's planner carries its own.
