# Memory convention

Shared contract for every `aw-*` skill that needs to remember or recall across
sessions. A skill declares **intent** — `persist` or `recall` — and this file
maps that intent to whatever mechanism the running agent actually has.

## Rules

1. **A skill NEVER names a memory tool.** Not in its prose, not in an example.
   Naming one couples the harness to a product; the harness is tool-agnostic by
   design. Skills say *"persist this digest"* / *"recall prior work on X"* — nothing more.
2. **Memory is ALWAYS scoped to a project.** Never global, never cross-repo. One
   rule, whatever the backend implements it with.
3. **Graceful degradation.** No memory available is not an error — the skill still
   does its job, it just skips the remember/recall step.

## Mechanism

The agent resolves intent to the first mechanism it has, in order:

```
persist(digest, project):
  1. A persistent-memory capability is available in the session
       → use it, scoped to `project`.
  2. Else → append to  .aw/memory/<project>/<topic>.md   (workspace root, gitignored)
  3. Else → skip.

recall(query, project):  symmetric — capability → files → none.
```

- **Capability** = any persistent-memory tool the agent has loaded (an MCP memory
  server, a native store, whatever). It carries its own project scoping — use its
  project field. The convention does not know or care which one it is.
- **File fallback** scopes by path: `.aw/memory/<project>/<topic>.md`. This mirrors
  the `.worktrees/<repo>/<task>/` namespacing — same shape, one level for project,
  the rest for the topic.

## Project resolution

```
project = the synced repo the work targets   →  repos/<name>  ⇒  <name>
          not tied to a repo                  →  _workspace
```

## What a skill persists

The **digest**, not the artifact. A skill that produces a full document (e.g.
`aw-research` writes `research.md` into the repo) keeps that file in the repo and
persists only a short, searchable digest through this convention — so downstream
skills can recall the gist without re-reading the whole file.
