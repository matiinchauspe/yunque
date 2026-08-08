# Flow convention

Shared contract for every `aw-*` skill that **walks** a charted map — moves through the
decision tickets an effort has been charted onto, rather than storing them. Where
`artifact-convention.md` holds the plan **at rest** — the map, its tickets, their types and
dependency edges — this contract governs the plan **in motion**: which ticket is takeable now,
who is working it, at what pace, and what runs unattended. A skill declares **intent** —
`frontier`, `claim`, or `release` — and follows the walk **rules** below; this file maps both
to whatever tracker the running agent has.

## Rules

1. **A skill NEVER names a tracker.** Not in its prose, not in an example. Naming one couples
   the harness to a product; the harness is tool-agnostic by design. Skills say *"claim this
   ticket"* / *"compute the frontier"* — nothing more.
2. **Scoped to a project.** The walk never crosses efforts or repos — one map at a time, one
   rule, whatever the backend implements it with.
3. **Concurrency is the backend's to enforce, not the filesystem's.** The absence of a tracker
   capability is not an error — the walk still proceeds. But only a capability backend can hold
   a claim, so only there is parallel walking safe. The file floor has no lock and no claim
   field, so walking one feature from two sessions at once is unsupported: the floor is
   single-writer, claiming goes mute, and the walk is serial.

## Mechanism

The agent resolves intent to the first mechanism it has, in order:

```
frontier(project, feature):  the takeable set — the edge of the known. A ticket is takeable
     when it is open, unblocked (every blocker in a terminal state — resolved or out-of-scope,
     never open — no longer gates), and unclaimed. Recompute it after every ticket closes —
     resolved or ruled out-of-scope: a ticket unblocked mid-walk re-enters. Each tier realizes
     that one predicate with its own primitives:
  1. A capability that renders its own frontier
       → query it directly: open children, no open blocker, no assignee, in its own order.
  2. Else → `list` the decision tickets through artifact-convention.md; a ticket is unblocked
         when every name on its `Blocked by:` line is terminal — join each named blocker to its
         own status in the `list` result. Keep the open, unblocked ones, first by number.
         The file ticket format carries no claim field, so "unclaimed" is vacuously true here —
         safe only because the floor is single-writer (Rule 3).

claim(ticket, project) / release(ticket, project):  the concurrency lock.
  1. capability → assign the ticket to the driving session; that assignee IS the claim (an
         open, unassigned ticket is unclaimed). release = unassign.
  2. Else → on the single-writer file floor there is nothing to race, so claim and release are
         no-ops: the one writer takes the first frontier ticket.
```

- **Claim first, release when the hold ends.** Claim before any work, so a concurrent session
  skips the ticket instead of duplicating it. Resolving the ticket closes it, which subsumes
  the claim; a session that ends with the ticket still open releases it, so it returns to the
  frontier. A capability backend has no liveness signal, so a claim a crashed session never
  released cannot be auto-reclaimed — it surfaces as a stranded claim on an otherwise-empty
  frontier (see *Empty frontier*), for a human to release.
- **Capability** = any issue-tracker tool the agent has loaded — the same one
  `artifact-convention.md` resolves against. It carries its own project scoping and, often,
  its own frontier view and assignee model; use them.
- **File fallback** reads the tickets `artifact-convention.md` wrote under
  `.aw/artifacts/<project>/<feature>/` — flow computes over that store, it does not keep one
  of its own.

## Walk rules

The plan in motion. These are pure discipline — no backend to resolve, they hold on every
tracker.

- **Cadence — one HITL ticket per session.** A human-in-the-loop decision spends a full
  context and a live exchange, so a session resolves at most one, then a fresh session
  re-orients on the changed map. AFK work does not count against this — no human is spent.
- **Autonomy — follow the HITL/AFK axis.** Every ticket's type carries whether a human is in
  the loop — fixed by type for research (AFK), prototype and grilling (HITL), and stamped on
  the ticket for a `task`, which is either. A **HITL** ticket resolves only through the live
  human exchange and is never auto-resolved — the agent never stands in for the human's side of
  it. An **AFK** ticket is safe to drive unattended: a research ticket is auto-dispatched
  (below); any other AFK ticket (a `task` stamped AFK) the walk drives inline, serially.
- **Auto-dispatch — research is the one parallel loop.** Fire a research subagent per research
  ticket *in the frontier*, in parallel — a blocked research ticket waits like any other.
  Research is AFK and does not block the cadence: its finding lands on the ticket at resolution
  and unblocks whatever depended on it. No other type is auto-dispatched to a subagent; HITL
  types stop at the human.
- **Push right — batch the human's questions.** Don't interrupt at the first HITL ticket. Drive
  every frontier AFK ticket first — dispatch research, run AFK tasks inline, recomputing as
  resolutions land; a resolved HITL may free AFK work, so drain that in the same session — the
  cadence limits only the human's turns, not the AFK drain. When only HITL tickets remain,
  surface them together, once, late, with the AFK findings already prepared. Surfacing does not
  claim — only the ticket the human picks is claimed, and the human resolves one; the rest wait
  for fresh sessions. If inline AFK exhausts the session before the frontier drains, surface
  progress and hand to a fresh session.
- **Empty frontier — read why.** No open tickets remain: the course is cleared, the walk is
  done, hand the map off. Open tickets remain but none is takeable: surface the blocking graph
  (the `Blocked by:` edges and statuses `list` returns) — the walk is stuck, not done; a ticket
  held only by a claim from a crashed session is released by hand here. No map at all: the
  effort was never charted, there is nothing to walk.

## Scope resolution

`project` and `<feature>` resolve exactly as in `artifact-convention.md` — the walk moves over
the same tracked-work artifacts that convention stores.

## Ownership

Flow owns **takeability and concurrency** — which ticket is takeable now and who holds it. The
ticket's *resolution content* — the decision it records — is owned by the skill its type
dispatches to, not by this contract.
