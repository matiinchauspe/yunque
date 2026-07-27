# Artifact convention

Shared contract for every `aw-*` skill that records or fetches **tracked work** across
sessions — the tickets a plan is sliced into. A skill declares **intent** — `publish` or
`fetch` — and this file maps that intent to whatever tracker the running agent actually has.

## Rules

1. **A skill NEVER names a tracker.** Not in its prose, not in an example. Naming one
   couples the harness to a product; the harness is tool-agnostic by design. Skills say
   *"publish this ticket"* / *"fetch the referenced ticket"* — nothing more.
2. **Artifacts are ALWAYS scoped to a project.** Never global, never cross-repo — one rule,
   whatever the backend implements it with.
3. **Graceful degradation, but the work always lands.** Unlike a `fetch`, publishing a
   ticket is the deliverable — it can never be skipped. When no tracker capability is present
   the floor is local files, which always work.

## Mechanism

The agent resolves intent to the first mechanism it has, in order:

```
publish(ticket, project, feature):
  1. An issue-tracker capability is available in the session
       → use it, scoped to `project`. Draw the ticket's edges with the tracker's
         own blocking / sub-issue relationship where it has one.
  2. Else → write  .aw/artifacts/<project>/<feature>/<NN>-<slug>.md   (workspace root, gitignored),
         numbered from 01 in dependency order, edges as a `Blocked by:` line.

fetch(reference, project):  symmetric — capability → files → not found. The reference names the ticket's feature and number.
```

- **Capability** = any issue-tracker tool the agent has loaded (an MCP tracker server, a
  CLI, a native store, whatever). It carries its own project scoping — use its project field.
  The convention does not know or care which one it is.
- **File fallback** scopes by path: `.aw/artifacts/<project>/<feature>/`, sharing the
  `.aw/<kind>/<project>/` namespace with `.aw/memory/` and nesting a numbered ticket file
  beneath the feature.

## Scope resolution

`project` resolves exactly as in `memory-convention.md`. `<feature>` is the slug of the work
being sliced — the plan or spec's title.

## Ownership

A ticket's **content** is backend-independent: the skill that produces it owns what it says;
this convention owns only where it lands and how its edges are drawn.
