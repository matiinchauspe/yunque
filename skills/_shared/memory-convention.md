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
persist(digest, topic, project):
  1. .yun/memory/<project>/index.md exists → the wiki.
  2. A persistent-memory capability is available in the session AND can scope to
     `project` → use it, scoped to `project`.
  3. Else → the wiki, creating it.   (.yun/memory/<project>/, workspace root, gitignored)
  4. Else → skip.

recall(query, project):  index.md exists → the wiki; else capability; else the project's
                         flat .md notes, if any; else none.
```

- **Capability** = any persistent-memory tool the agent has loaded (an MCP memory
  server, a native store, whatever). It carries its own project scoping — use its
  project field, and **confirm the scope it lands on is `project`** — one that infers its
  scope from the working directory resolves to the *workspace*, since AGENTS.md rule 1
  keeps every session at the root. A scope you cannot set to `project` is a **failed
  step 2**: fall through to the wiki, which scopes by path. The convention does not know
  or care which capability it is.
- **An empty `recall` is not proof of absence.** A capability can miss what it holds: a long,
  specific query returns nothing where one short distinctive term returns the answer. Retry
  with the shortest distinctive term before concluding nothing is stored.

## The wiki

The file rung is a small **wiki**, after Karpathy's LLM wiki: raw digests in, compiled pages out.
Flat, in `.yun/memory/<project>/`:

- `raw.md` — every persisted digest, whole, under `## [YYYY-MM-DD] <topic> | <title>`.
  Append-only: never edited, never read whole on recall.
- `<topic>.md` — one page per topic: what holds today, compiled from `raw.md`.
- `index.md` — one line per page: `- [<topic>](<topic>.md) — <summary> · <YYYY-MM-DD>`.

**persist** is one step: append to `raw.md`, rewrite the topic's page to what holds now, fix its
`index.md` line. A superseded claim leaves the page; it stays in `raw.md`.
**recall** reads `index.md`, then the matching page; no page matches → search `raw.md`.

`index.md` is the switch, and it **sticks**: a project that has one stays on the wiki even with a
capability loaded — one home per project, so nothing filed while the capability was missing turns
orphan when it returns. A persist that reaches step 3 creates it, and so does the user asking to
switch a project on. Creating a new `index.md` first ingests into `raw.md` what the project already
holds: its flat `.md` notes, dated by file mtime — one entry per dated `##` section when a note has
them — and the capability's records scoped to `project`, dated by creation. A `handoff-*.md` is a
baton, not memory: it stays out. Then compile their pages and tell the user in one line that the
project switched to wiki memory. Deleting `index.md` switches it back; when the user asks for that,
confirm first, naming what it costs: the pages stop being read, new notes go to the capability —
and with none, the next persist switches the wiki back on — and switching on again later brings in
nothing filed meanwhile. A project that already holds `raw.md` is switching on again, so rebuild
`index.md` from its pages and ingest nothing.

**lint** runs only when the user asks. A persist touches one page, so only lint sees across pages.
It fixes the mechanical — every page has its `index.md` line and every line its page; a referenced
path that no longer exists gets marked — and reports, without fixing, the judgment calls: pages
that contradict each other, a page claim `raw.md` does not back.

## Project resolution

```
project = the synced repo the work targets   →  repos/<name>  ⇒  <name>
          the harness itself                  →  the workspace repo's own name
          tied to no repo at all              →  _workspace
```

**The subject decides the project, not the session.** A note about the harness itself — a skill
that misfired, a contract that missed the case, a step done by hand — resolves to the harness even
when you noticed it mid-project. Persist it under the fixed topic `friction` and carry on: raising
it spends the user's turn on a problem they did not come to work, and filing it under their project
buries it.

`friction` accumulates, so **recall it and write it back with yours added** — never persist your
note alone. A capability may REPLACE what is under a topic, so one blind write erases every note
before it. Holds for any topic read as a running list rather than a latest-value.

## What a skill persists

The **digest**, not the artifact. A skill that produces a full document (e.g.
`yun-research` writes `research.md` into the repo) keeps that file in the repo and
persists only a short, searchable digest through this convention — so downstream
skills can recall the gist without re-reading the whole file.
