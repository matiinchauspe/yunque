---
name: aw-chart-course
description: Chart a course through work too big for one session — a live map of decision tickets, resolved one at a time until the way to the destination is clear.
disable-model-invocation: true
---

# aw-chart-course

**Plan, don't do.** Some work is too big for one session and wrapped in **fog** — the way
from here to the **destination** isn't visible yet. Charting is finding that way, not
charging at it: each **decision ticket** resolves one open question, and the course is clear
when nothing is left to decide before someone goes and builds. The pull to just write the
code is the signal you've reached the edge of the map — hand off, don't build.

## Guard — is this even a charting job?

The heaviest flow in the harness. Reserve it for the idea that genuinely won't fit one
session. Two checks before you chart:

- **Size.** A well-scoped feature — one that fits a single session — is not a charting job.
  Hand it to `aw-grill-plan` → `aw-write-spec`, not here.
- **Fog.** Name the destination and grill it breadth-first. If no fog surfaces — you can
  already state every decision — you don't need a map. Stop and hand to `aw-write-spec`.

Done when the work is confirmed too big for one session **and** fog remains after naming the
destination.

## The map

One live artifact per effort, `publish`ed as the `map` kind through
`skills/_shared/artifact-convention.md`. It is an **index, not a store**: a decision lives in
exactly one place — its ticket — and the map only gists it and links. Refer to every ticket
by its title, never a bare id. The body is the whole course at low resolution, read once at
the start of every session.

<map-template>

## Destination
<what reaching the end looks like — the spec, decision, or change this effort finds its way to. One or two lines; every session orients to it first.>

## Notes
<domain; skills every session should consult; standing preferences for this effort>

## Decisions so far
<the index — one line per resolved ticket, enough to judge relevance, then zoom the link>
- [<resolved ticket title>](link) — <one-line gist of the answer>

## Not yet specified
<in-scope fog you can't ticket yet; graduates as the frontier advances>

## Out of scope
<work ruled beyond the destination; ruled out, never graduates>

</map-template>

## Decision tickets

Each ticket is a child of the map, its body one **Question** — the decision it resolves,
sized to a single session. It carries a `Type:` that says which skill resolves it and whether
a human is in the loop:

- **research** (AFK) — surface a fact a decision waits on, from docs or APIs outside the
  working directory. Resolved by an `aw-research` subagent.
- **prototype** (HITL) — make a cheap concrete artifact to react to when "how should it look
  / behave" is the question. Resolved by `aw-build-prototype`.
- **grilling** (HITL) — one question at a time via `aw-grill-plan`, paired with
  `aw-model-domain` when vocabulary is at stake. The default type.
- **task** (HITL or AFK) — manual work that must happen before a decision can be made
  (provisioning access, moving data so its shape is visible). The one type that *does* rather
  than decides; resolved when the work is done and its resulting facts are recorded.

**HITL** means the decision resolves only through a live exchange — the agent never stands in
for the human's side of it. Resolve tickets one at a time.

### Fog or ticket?

The test is whether you can state the question precisely now — *not* whether you can answer
it. **Ticket** when it's already sharp, even if blocked. **Not yet specified** when you can't
phrase it that sharply yet — don't pre-slice fog into ticket-sized pieces; one patch may
graduate into several tickets, or none. Fog only ever gathers *toward* the destination; work
beyond it is **Out of scope**, and never graduates.

## Chart the map

1. **Name the destination.** Grill it with `aw-grill-plan` (and `aw-model-domain` for
   vocabulary). Naming it first shapes every ticket. Done when the destination is one or two
   lines you can publish.
2. **Map the frontier.** Grill breadth-first for the decisions between here and there. Apply
   the fog-or-ticket test to each. Done when every surfaced question is either a sharp
   ticket-to-be or a line of fog.
3. **Publish the map, then the tickets.** `publish` the `map` kind, then `publish` each
   specifiable ticket with its `Type:`. Wire blocking edges once every ticket has an identity
   (on a backend that mints ids on creation, that means a second pass). Done when the map and
   every open ticket exist and every blocking edge is drawn.
4. **Stop.** Charting is one session's work; it resolves nothing. Hand off — the next session
   works the map.

## Work through the map

1. **Orient.** `fetch` the `map` kind through `skills/_shared/artifact-convention.md` and load
   it at low resolution — but the map indexes only closed tickets, so `list` the effort's
   decision tickets through the convention to see the open ones. From that list compute the
   **frontier** — the open tickets none of whose blockers are still open (a resolved or
   out-of-scope blocker no longer gates), in the backend's own order — and print it (a backend
   that renders its own frontier makes this a harmless echo). Done when you know which tickets
   are takeable.
2. **Take one ticket.** The user's choice, or the first frontier ticket. Dispatch it to the
   skill its `Type:` names; resolve it there, zooming into the map only as needed. Resolve one
   ticket per session — each ticket spends a full context, and the file floor is single-writer.
3. **Record the resolution.** Post the answer to the ticket, mark it resolved, and append a
   one-line gist + link to the map's **Decisions so far**. Done when the resolved ticket is
   indexed on the map.
4. **Graduate the fog.** A resolved ticket clears the fog ahead of it: promote whatever is now
   sharp from **Not yet specified** into fresh tickets, and rescope anything that turned out
   to sit past the destination into **Out of scope**. Done when the map reflects the new
   frontier.

When no tickets remain, the way is clear: hand the cleared map to `aw-write-spec`, which
collapses its decisions into a buildable spec. Chart, then spec — never loop the map straight
into `aw-implement`.

## Attribution

Adapted for this workspace from **Matt Pocock's `wayfinder`** (github.com/mattpocock/skills,
Apache-2.0). Changes: renamed to the `aw-<verb>-<noun>` convention; the map is published as a
new `map` artifact kind and all tracker mechanics (native dependencies, `gh` API, sub-issues)
are pushed behind `skills/_shared/artifact-convention.md`, so the skill never names a tracker;
the four ticket types dispatch to the harness siblings `aw-research` / `aw-build-prototype` /
`aw-grill-plan` + `aw-model-domain`, and the cleared map hands off to `aw-write-spec`. The
plan-don't-do spine, fog-of-war, decision-ticket typing, and HITL/AFK axis are preserved from
the source; the source's one-ticket-per-session rule is preserved but tightened — its research
exception folds into the serial default until a future `flow-convention` restores it. Parallel
research subagents, ticket claiming, and push-right question batching are deferred to that
`flow-convention`.
