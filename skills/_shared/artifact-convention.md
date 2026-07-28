# Artifact convention

Shared contract for every `aw-*` skill that records or fetches **tracked work** across
sessions — the two artifact **kinds** a plan produces: the **spec** it is written into and
the **tickets** it is sliced into. A skill declares **intent** — `publish` or `fetch` — and
this file maps that intent to whatever tracker the running agent actually has.

## Rules

1. **A skill NEVER names a tracker.** Not in its prose, not in an example. Naming one
   couples the harness to a product; the harness is tool-agnostic by design. Skills say
   *"publish this spec"* / *"publish this ticket"* / *"fetch the referenced artifact"* —
   nothing more.
2. **Artifacts are ALWAYS scoped to a project.** Never global, never cross-repo — one rule,
   whatever the backend implements it with.
3. **Graceful degradation, but the work always lands.** Unlike a `fetch`, publishing a spec
   or a ticket is the deliverable — it can never be skipped. When no tracker capability is
   present the floor is local files, which always work.

## Mechanism

The agent resolves intent to the first mechanism it has, in order:

```
publish(artifact, project, feature):        # artifact is a spec or a ticket
  1. An issue-tracker capability is available in the session
       → use it, scoped to `project`. A spec becomes the feature's spec document;
         a ticket becomes an issue, its edges drawn with the tracker's own
         blocking / sub-issue relationship where it has one.
  2. Else → write under  .aw/artifacts/<project>/<feature>/   (workspace root, gitignored):
       - a spec   → spec.md                (one per feature)
       - a ticket → <NN>-<slug>.md         numbered from 01 in dependency order,
                                           edges as a `Blocked by:` line.

fetch(reference, project):  try each mechanism until one YIELDS the artifact — a present but
     empty capability falls through, it does not short-circuit:
       1. capability → address a spec by the feature key, a ticket by feature + number.
       2. files      → read a spec at .aw/artifacts/<project>/<feature>/spec.md;
                       a ticket at the numbered file the reference names.
       3. neither yields it → if the artifact is in the conversation, use that; otherwise
          say the referenced artifact is unreachable and stop — never proceed on nothing.
     The reference names the artifact's feature, plus the ticket number when it is a ticket.
```

- **Capability** = any issue-tracker tool the agent has loaded (an MCP tracker server, a
  CLI, a native store, whatever). It carries its own project scoping — use its project field.
  The convention does not know or care which one it is.
- **File fallback** scopes by path: `.aw/artifacts/<project>/<feature>/`, sharing the
  `.aw/<kind>/<project>/` namespace with `.aw/memory/` and nesting a numbered ticket file
  beneath the feature.

## Scope resolution

`project` resolves exactly as in `memory-convention.md`. `<feature>` is one canonical slug:
the spec's title where a spec exists, else the plan's title. `publish` and every later
`fetch` slugify that same title, so the spec written and the spec fetched resolve to one key.
A feature title is assumed unique within a project — it *is* the key; two features must not
share one, or the second `publish` overwrites the first.

## Ownership

An artifact's **content** — spec or ticket — is backend-independent: the skill that produces
it owns what it says; this convention owns only where it lands and how its edges are drawn.
