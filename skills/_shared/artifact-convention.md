# Artifact convention

Shared contract for every `aw-*` skill that records or fetches **tracked work** across
sessions — the artifact **kinds** a plan produces: the **map** an effort is charted onto, the
**spec** it is written into, and the **tickets** it breaks into (decision tickets when
charted, implementation tickets when sliced). A skill declares **intent** — `publish`,
`fetch`, `list`, or `resolve` — and this file maps that intent to whatever tracker the running
agent has.

## Rules

1. **A skill NEVER names a tracker.** Not in its prose, not in an example. Naming one
   couples the harness to a product; the harness is tool-agnostic by design. Skills say
   *"publish this spec"* / *"publish this ticket"* / *"fetch the referenced artifact"* /
   *"list the tickets in this namespace"* / *"resolve this ticket"* — nothing more.
2. **Artifacts are ALWAYS scoped to a project.** Never global, never cross-repo — one rule,
   whatever the backend implements it with.
3. **Graceful degradation, but the work always lands.** Unlike a `fetch`, publishing a spec
   or a ticket is the deliverable — it can never be skipped. When no tracker capability is
   present the floor is local files, which always work.

## Mechanism

The agent resolves intent to the first mechanism it has, in order:

```
publish(artifact, project, feature):        # artifact is a map, a spec, or a ticket
  1. An issue-tracker capability is available in the session
       → use it, scoped to `project`, leaning on its NATIVE mechanics — blocking, ordering,
         and terminal state are the tracker's own; the convention does not re-specify them.
         A map is the effort's canonical issue (its tickets its children), found-or-created by
         feature key and edited in place across sessions — never a second issue; a spec becomes
         the feature's spec document; a ticket becomes an issue, its edges drawn with the
         tracker's own blocking / sub-issue relationship.
         **Every ticket is scoped to its feature by a marker the backend can query — the
         feature key — whether or not a map exists.** Where the effort was charted, the map is
         also its parent; where it was not, the feature key alone scopes it. A ticket is never
         reachable only through a map, or an unmapped effort would be invisible to `list`.
         **Each ticket also carries a marker naming its namespace, decision or implementation
         — stated positively on both, never inferred from the absence of the other**, so a
         child issue some human or other tool created is not swept into either set.
         A decision ticket additionally carries its type as a label and blocks only other
         decision tickets; a ticket ruled beyond the destination is closed and marked
         out-of-scope the tracker's own way (a label or resolution reason) —
         closed-as-out-of-scope, not resolved. An implementation ticket blocks only other
         implementation tickets, and nothing rules it out-of-scope. Either way the ticket's own
         number leads its title, so it stays addressable whatever number the tracker assigns.
         **The publishing skill assigns that number** — the next free one in that namespace
         under this feature, read from `list` — because a tracker's own id is the tracker's,
         and edges drawn on it would not survive a move to another backend.
  2. Else → write under  .aw/artifacts/<project>/<feature>/   (workspace root, gitignored):
       - a map    → map.md                 (one per feature; a live index, edited in place
                                           across sessions — appended to, never regenerated
                                           from scratch. Its links to decision tickets are
                                           their relative paths, `decisions/<NN>-<slug>.md`)
       - a spec   → spec.md                (one per feature)
       - a decision ticket        → decisions/<NN>-<slug>.md   numbered from 01, edges as a
                                    `Blocked by:` line naming other decision tickets (same
                                    namespace), with a `Type:` line (research / prototype /
                                    grilling / task — a `task` records its HITL/AFK mode, e.g.
                                    `task (AFK)`) and a `Status:` line (open / resolved /
                                    out-of-scope). Updated in place — resolving flips Status to
                                    resolved, rescoping past the destination to out-of-scope;
                                    only `open` tickets enter the frontier.
       - an implementation ticket → <NN>-<slug>.md            numbered from 01 in dependency
                                    order (an independent counter from the decision-ticket
                                    one), edges as a `Blocked by:` line naming other
                                    implementation tickets (same namespace), and a `Status:` line
                                    (open / resolved), flipped in place through `resolve` below.
                                    WHEN a ticket has earned `resolved` is the walking skill's
                                    call, never this contract's. Nothing here is ruled
                                    out-of-scope: scope is settled while charting, not while
                                    building.

fetch(reference, project):  try each mechanism until one YIELDS the artifact — a present but
     empty capability falls through, it does not short-circuit:
       1. capability → address a map or spec by the feature key, a ticket by feature + number.
       2. files      → read a map at .aw/artifacts/<project>/<feature>/map.md, a spec at
                       spec.md; a decision ticket at decisions/<NN>-<slug>.md, an
                       implementation ticket at the numbered file the reference names.
       3. neither yields it → if the artifact is in the conversation, use that; otherwise
          say the referenced artifact is unreachable and stop — never proceed on nothing.
     The reference names the artifact's feature, plus the ticket number — and, for a ticket,
     whether it is a decision or implementation ticket.

list(project, feature, namespace):  enumerate a feature's tickets in ONE namespace — decision
     or implementation, never both. They are two separate sets on independent counters, so a
     caller asks for the one it walks. (The map indexes only closed tickets, so open ones are
     found here, not on the map.)
       1. capability → query by the FEATURE KEY and the namespace marker, scoped to
                       `project` — never by walking a map's children, since an effort that was
                       sliced without being charted has no map and its tickets must still be
                       found. Read each result's number, blocking edges, terminal state, and —
                       for decision tickets — its type, the tracker's own way.
       2. files      → scan the namespace's own path: decision tickets under
                       .aw/artifacts/<project>/<feature>/decisions/ , implementation tickets at
                       .aw/artifacts/<project>/<feature>/<NN>-<slug>.md . Read each ticket
                       file's `Blocked by:` and `Status:` lines, plus `Type:` where the
                       namespace carries one.
     Returns every ticket in that namespace with its NUMBER, edges, status, and — where the
     namespace carries one — its `Type:`, which the decision walk's rules key off. All statuses,
     so the caller can see that a terminal blocker no longer gates. **The number is the join
     key**: a `Blocked by:` entry names the blocker's number, so edges resolve without matching
     titles. It is empty only when the feature has no tickets in that namespace at all. What
     the returned set means for a walk — which tickets are takeable, and what an empty result
     implies — belongs to the caller: `flow-convention.md` for the decision namespace, the
     dispatching skill for the implementation one. The walk lives there, not here.

resolve(reference, project, state):  move ONE ticket to a terminal state — `resolved`, or
     `out-of-scope` where its namespace allows it. The reference resolves as in `fetch`. This
     contract owns only that the state lands and is readable by a later `list`; the judgment
     that a ticket has EARNED it belongs to the skill that walks that namespace.
       1. capability → close the ticket the tracker's own way, marking out-of-scope with its
                       own label or resolution reason where that is the state.
       2. files      → rewrite the ticket file's `Status:` line in place, leaving the rest of
                       the file untouched.
     A ticket already in the asked-for state is left alone — resolving twice is not an error.
```

- **Capability** = any issue-tracker tool the agent has loaded (an MCP tracker server, a
  CLI, a native store, whatever). It carries its own project scoping — use its project field.
  The convention does not know or care which one it is.
- **File fallback** scopes by path: `.aw/artifacts/<project>/<feature>/`, sharing the
  `.aw/<kind>/<project>/` namespace with `.aw/memory/` and nesting a numbered ticket file
  beneath the feature.

## Scope resolution

`project` resolves exactly as in `memory-convention.md`. `<feature>` is one canonical slug:
for a charted effort, the map's destination slug, fixed when the map is first published and
reused verbatim by the spec and every ticket that follows, so the whole chain resolves to one
key — a charted effort's key is NOT re-derived from the spec's title. Otherwise it is the
spec's title where a spec exists, else the plan's title. `publish` and every later `fetch`
slugify that same title, so the artifact written and the artifact fetched resolve to one key.
A feature title is assumed unique within a project — it *is* the key; two features must not
share one, or the second `publish` overwrites the first.

## Ownership

An artifact's **content** — map, spec, or ticket — is backend-independent: the skill that
produces it owns what it says; this convention owns only where it lands, how its edges are
drawn, and that a terminal state persists — never WHEN a ticket earns one. The tickets `list`
returns are the authoritative record in either namespace; the map's Decisions-so-far index is a
derived, human-readable view — if the two diverge, repair the index, not the tickets.
