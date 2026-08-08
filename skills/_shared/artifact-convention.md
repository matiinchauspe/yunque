# Artifact convention

Shared contract for every `aw-*` skill that records or fetches **tracked work** across
sessions — the artifact **kinds** a plan produces: the **map** an effort is charted onto, the
**spec** it is written into, and the **tickets** it breaks into (decision tickets when
charted, implementation tickets when sliced). A skill declares **intent** — `publish`,
`fetch`, or `list` — and this file maps that intent to whatever tracker the running agent has.

## Rules

1. **A skill NEVER names a tracker.** Not in its prose, not in an example. Naming one
   couples the harness to a product; the harness is tool-agnostic by design. Skills say
   *"publish this spec"* / *"publish this ticket"* / *"fetch the referenced artifact"* /
   *"list the decision tickets"* — nothing more.
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
         tracker's own blocking / sub-issue relationship. A decision ticket carries its type as
         a label and blocks only other decision tickets; resolving it closes it, and a ticket
         ruled beyond the destination is closed and marked out-of-scope the tracker's own way
         (a label or resolution reason) — closed-as-out-of-scope, not resolved.
  2. Else → write under  .aw/artifacts/<project>/<feature>/   (workspace root, gitignored):
       - a map    → map.md                 (one per feature; a live index, edited in place
                                           across sessions — appended to, never regenerated
                                           from scratch. Its links to decision tickets are
                                           their relative paths, `decisions/<NN>-<slug>.md`)
       - a spec   → spec.md                (one per feature)
       - a decision ticket        → decisions/<NN>-<slug>.md   numbered from 01, edges as a
                                    `Blocked by:` line naming other decision tickets (same
                                    namespace), with a `Type:` line (research / prototype /
                                    grilling / task) and a `Status:` line (open / resolved /
                                    out-of-scope). Updated in place — resolving flips Status to
                                    resolved, rescoping past the destination to out-of-scope;
                                    only `open` tickets enter the frontier.
       - an implementation ticket → <NN>-<slug>.md            numbered from 01 in dependency
                                    order (an independent counter from the decision-ticket
                                    one), edges as a `Blocked by:` line naming other
                                    implementation tickets (same namespace).

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

list(project, feature):  enumerate a feature's decision tickets — the raw material a frontier
     is computed from (the map indexes only closed tickets, so open ones are found here, not on
     the map):
       1. capability → resolve the map by the feature key, then query its children scoped to
                       `project`, reading each child's type, blocking edges, and terminal state
                       (resolved vs out-of-scope) the tracker's own way.
       2. files      → scan .aw/artifacts/<project>/<feature>/decisions/ , reading each ticket
                       file's `Type:`, `Blocked by:`, and `Status:` lines.
     Returns every decision ticket with its type, edges, and status — all statuses, so the
     caller can see that a resolved or out-of-scope blocker no longer gates. It is empty only
     when the effort has no decision tickets at all. The caller has already `fetch`ed the map,
     so it reads the signals apart: a map with no OPEN tickets in the returned set is a cleared
     course; no map means the effort was never charted. From the returned set the caller
     computes the frontier — the open tickets none of whose blockers are still open (resolved or
     out-of-scope no longer gates) — in the backend's own order.
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
produces it owns what it says; this convention owns only where it lands and how its edges are
drawn. For the frontier, the decision tickets `list` returns are authoritative; the map's
Decisions-so-far index is a derived, human-readable view — if the two diverge, repair the
index, not the frontier.
