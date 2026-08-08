# Agentic Workspace

A **project-agnostic workspace**: an umbrella directory, with its own repo, that hosts
your agent harness (skills, rules, flows) decoupled from any single project. You bring
whatever repo you want inside and work it with YOUR harness — not the other way around.

## Core idea

The harness rules. Projects are interchangeable work material that comes and goes.
This is dependency inversion: the project depends on the workspace, never the workspace
on a project.

## How to use it (the flow)

1. Open your agent from the **root** of this workspace → the harness loads and rules.
2. **Bring a repo in:** `aw-sync-repo <url|name>` → clones into `repos/<name>/`
   (gitignored, its own `.git`).
3. **Work it with the harness.** Small and settled → `aw-implement` builds it test-first
   (`aw-tdd`), then `aw-review-work` stress-tests it. Large or foggy → plan first:
   `aw-grill-plan` → `aw-write-spec` → `aw-slice-plan`, then build. Too big to hold in one
   session → `aw-chart-course` charts it as a map of decisions first, then hands off to that
   planning chain.
4. **Parallelize** when you need to: `aw-spawn-worktree <repo> <task>` → an isolated agent
   in `.worktrees/<repo>/<task>/` on its own branch. When done, `git worktree prune`.

## Current state

The harness is operational — 14 skills over four shared contracts: bring-in
(`aw-sync-repo`), charting (`aw-chart-course`), planning (`aw-grill-plan` → `aw-write-spec` →
`aw-slice-plan`), build (`aw-implement`, `aw-tdd`), knowledge (`aw-research`,
`aw-model-domain`), prototyping (`aw-build-prototype`), isolation (`aw-spawn-worktree`),
adversarial review (`aw-review-work`), handoff (`aw-write-handoff`), and authoring
(`aw-write-skill`). Contracts: `memory-`, `artifact-` (map/spec/ticket kinds, the plan at
rest), `domain-`, and `flow-convention` (walking a charted map — the plan in motion).

Deferred: a heavier execution substrate behind an `execution-convention` — autonomous
sandboxed runs that loop to completion — with `aw-spawn-worktree` as its lighter rung.

The harness brain is `AGENTS.md`; the skill index is `skills/README.md`; research notes
live in `docs/research/`.
