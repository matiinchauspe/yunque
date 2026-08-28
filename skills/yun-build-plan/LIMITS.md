# The walk's declared limits

Stated here rather than discovered: one shape the walk **refuses to build**, one judgment it
**cannot make and proceeds anyway**, and the places where the mechanism is weaker than it looks —
what a green gate, a chained branch, a resolved ticket, and a readable chain each fail to guarantee.

## What this walk does not build

**A ticket whose body carries an `Integrates on:` line naming a branch is declined.** Those are the
expand–contract batches `yun-slice-plan` sanctions for a wide refactor: they promise green only at a
downstream integrate-and-verify ticket, so each one is red by design.

**Only a named branch counts.** `yun-slice-plan` writes that line on batches alone and keeps it out
of its template on purpose, so an ordinary ticket carries no line at all — one left empty or filled
with `None` can only have come from a hand-edited ticket or another tool. Reading mere presence as a
batch would decline every ticket of an ordinary plan, so the walk reads the branch, not the line.

This walk's gate is whole-tree and its chain is serial, so a red link would leave every ticket after
it reading red — and the next innocent ticket would be reported as bad slicing, a finding about work
that is fine. Building the batch properly means a second chain the walk cuts, bases runs against,
and freezes its frontier on: a mechanism whose only customer is the rarest shape the slicer
produces. So the walk declines that ticket in the open and carries on — **declined, not fatal**. A
batch at ticket 03 of 12 would otherwise strand the nine behind it.

**What that leaves, when a plan contains a wide refactor, is a green deliverable that is half
migrated.** The expand ticket carries no `Integrates on:` line, so the walk builds it; the batches
are declined; the contract ticket strands behind them. The result gates green while carrying the new
form beside the old with no call site moved across — green means less here than anywhere else in the
walk. Step 4 reports the declines and the strand, so it is discoverable, but a human reads that
report to know it.

## Why nothing is filtered before dispatch

Of the three fitness criteria `skills/_shared/execution-convention.md` names, two are settled by
construction — takeable is the frontier the walk just computed, branch-producing is what a tracer
bullet is.

**Unattended is an assumption, not a proof.** Implementation tickets carry no axis on which a human
could mark one as needing them, so the walk cannot tell a ticket that wants a human from one that
does not — it proceeds as if none does. In a repo with runnable gates that is cheap: the gate
answers. In a repo with none, a ticket whose acceptance is really a judgement call can reach
`resolved` on a completion report alone. It is why step 4 reports the gate's absence, and it is what
an unattended review layer would close.

Declining the batch is the honest counterpart: where the walk cannot tell whether it should proceed,
it proceeds and says so; where it can tell it should not, it stops.

## The whole-tree verdict is the run's own word

The walk chains the next ticket onto a branch on a verdict the run reported about that branch. That
is deliberate — the run's workspace is the only place the project's environment stands up, and a
gate run in a bare checkout sees tracked files and no installed dependencies, so its red would be a
lie about the slicing. It also means **the run is judging the work it just did**.

What keeps that honest is that the verdict is mechanical: the repo's own test, build and typecheck
entry points, pass or fail. What it cannot survive is a run that skips them and says green anyway,
and nothing downstream re-checks. The deliverable is still a human's to review, which is where a lie
surfaces; but it surfaces late.

Everything the walk *can* read, it reads rather than trusting: whether the base it handed out is an
ancestor of what came back, whether anything was committed on top of it, whether the branch carries
the name it was given, and whether the host working tree is clean before a dispatch. A lied-about
gate surfaces at the human's review; a chain quietly cut from the wrong place never surfaces at all.

Closing the gate half means a check that holds the environment without belonging to the run. Until
then, a walk's `resolved` means *the run said its gates passed, and the branch it left provably
descends from what it was given and carries work of its own* — no more.

## A red gate stops everything, and that blast radius is the price

Every other untrustworthy outcome is a skip: the chain does not advance and the frontier carries on.
A red whole-tree gate is the one verdict that halts everything, because it is the one that can be
evidence about work **already chained** — the defect may have ridden in several tickets ago, with
only this ticket's gates sharp enough to expose it, and skipping would cut every later run from that
same suspect chain.

The cost falls on plans that are mostly fine: one bad slice at position 03 stops a walk whose
remaining nine tickets were buildable, and the human gets a report instead of a deliverable. The walk
accepts that rather than ship a defect it had already been shown.

## The gate never sees the mainline the work will land on

The walk reads the mainline once, for the first run's base — or picks up a chain already cut from
it — and never refreshes it. Every gate after that runs against a branch descended from that
snapshot, so the walk can only ever prove the work holds together **against the mainline as it stood
when the chain began**.

The longer the walk, the further that drifts. The deliverable is a branch a human merges, and that
merge is the first moment the assembled work meets what it actually lands on: it can conflict, and
it can go green here and red there. Every link *inside* the chain is cut from the last, so nothing
inside the walk can conflict — that is what being serial buys — but the merge *out* of it is an
ordinary merge with none of those guarantees.

The walk does not fetch, because a network call nobody asked for is its own hazard, and it does not
re-baseline mid-walk, because that would invalidate every gate it already ran. So it reports the
snapshot instead, and the human merging knows what the green did and did not cover.

## One linear branch makes rejecting a single ticket surgery

Chain-by-base buys integration by construction and pays for it in reviewability. The human receives
**one** branch, in which every ticket is an ancestor of the next, so declining ticket 09's work after
10 through 12 were built on top of it is a revert or a rewrite — not a branch dropped on the floor.
Under the source's fan-out model each ticket is a separately declinable branch, and that is a real
thing given up here, not an oversight.

Review is pushed right precisely because no human stands at the runs' convergence points; the cost
lands at the far end, where the granularity of rejection is the chain rather than the ticket.

## The chain is only as readable as the namespace, and only git is reversible

The walk keeps no state, so it survives any interruption — but only because `aw/<slug>/` and the
ticket statuses hold everything it would otherwise have remembered. Three things put that out of
reach: a **pruned namespace**; **two branches on one resolved number**, where no single branch holds
the others as ancestors; and a **tier that did not honour the name it was given**, which
`execution-convention:47-49` explicitly permits and which leaves a branch outside the namespace
entirely. Then step 1 stops and asks for `<base>`, which nothing can verify, so the walk proceeds on
a human's word and step 4 says so. The contract sanctions re-deriving a set from git alone; what it
cannot promise is that the refs are still there.

The control surface a human is left with is **asymmetric**, and that asymmetry decides what is
recoverable. Deleting a failed run's branch retries its ticket, and nothing about it is destructive —
git is the reversible half. `artifact-convention`'s `resolve` has no inverse: a ticket this walk
resolved reads `resolved` whether or not the human ever merges the deliverable, and a rejected
deliverable therefore leaves a feature whose every ticket claims to be built. A later walk over it
computes an empty frontier and reports the plan built over work that was thrown away. Rebuilding
means new tickets, or hand-editing statuses in a store the harness otherwise treats as
authoritative.

The source template earns free resumption a different way — deterministic branch names
(`plan-prompt.md:25`) — which trades against `yun-spawn-worktree` refusing to reuse a name, a real
decision rather than an oversight. Deriving from the namespace buys the same resumption without that
trade, and pays for it with the cases above.
