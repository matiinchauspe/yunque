# Memory convention

Shared contract for every `yun-*` skill that needs to remember or recall across
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
  1. A persistent-memory capability is available in the session AND can scope to
     `project` → use it, scoped to `project`.
  2. Else → append to  .yun/memory/<project>/<topic>.md   (workspace root, gitignored)
  3. Else → skip.

recall(query, project):  symmetric — capability → files → none.
```

- **Capability** = any persistent-memory tool the agent has loaded (an MCP memory
  server, a native store, whatever). It carries its own project scoping — use its
  project field, and **confirm the scope it lands on is `project`**. A capability that
  infers its scope from the working directory resolves to the *workspace*, because
  CLAUDE.md rule 1 keeps every session at the workspace root — so a project's memory
  silently files itself under the harness, and one namespace accumulates every
  project's work. A scope you cannot set to `project` is a **failed step 1**, not a
  step 1 with a wrong label: fall through to the files, which scope by path and cannot
  drift. The convention does not know or care which capability it is.
- **File fallback** scopes by path: `.yun/memory/<project>/<topic>.md`. This mirrors
  the `.worktrees/<repo>/<task>/` namespacing — same shape, one level for project,
  the rest for the topic.

## Project resolution

```
project = the synced repo the work targets   →  repos/<name>  ⇒  <name>
          the harness itself                  →  the workspace repo's own name
          tied to no repo at all              →  _workspace
```

**The subject decides the project, not the session.** A note about the harness itself — a skill
that misfired, a contract that did not cover the case, a step you had to do by hand — resolves to
the harness even when you noticed it in the middle of a project's work. Persist it under the fixed
topic `friction` and carry on. Both other roads are dead ends: raising it in the session spends the
user's turn on a problem that is not the one they came to work, and filing it under the project
they *are* working buries it where nobody maintaining the harness will ever look.

`friction` accumulates, so **recall it and write it back with yours added** — never persist your
note alone. The file floor appends and would keep both, but a capability may replace what is under
a topic, and a single blind write there erases every note before it. This holds for any topic read
as a running list rather than a latest-value.

## What a skill persists

The **digest**, not the artifact. A skill that produces a full document (e.g.
`yun-research` writes `research.md` into the repo) keeps that file in the repo and
persists only a short, searchable digest through this convention — so downstream
skills can recall the gist without re-reading the whole file.
