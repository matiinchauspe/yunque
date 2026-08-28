# Yunque

A **project-agnostic workspace**: an umbrella directory, with its own repo, that hosts
your agent harness (skills, rules, flows) decoupled from any single project. You bring
whatever repo you want inside and work it with YOUR harness — not the other way around.

## Core idea

The harness rules. Projects are interchangeable work material that comes and goes.
This is dependency inversion: the project depends on the workspace, never the workspace
on a project.

## How to use it (the flow)

1. Open your agent from the **root** of this workspace → the harness loads and rules.
2. **Bring a repo in:** `yun-sync-repo <url|name>` → clones into `repos/<name>/`
   (gitignored, its own `.git`).
3. **Work it with the harness.** Small and settled → `yun-implement` builds it test-first
   (`yun-tdd`), then `yun-review-work` stress-tests it. Large or foggy → plan first:
   `yun-grill-plan` → `yun-write-spec` → `yun-slice-plan`, then `yun-build-plan` walks the sliced
   tickets to done and leaves one branch to review. Too big to hold in one
   session → `yun-chart-course` charts it as a map of decisions first, then hands off to that
   planning chain.
4. **Parallelize** when you need to: `yun-spawn-worktree <repo> <task>` → an isolated agent
   in `.worktrees/<repo>/<task>/` on its own branch. When done, `git worktree prune`.

## Current state

The harness is operational — 15 skills over five shared contracts: bring-in
(`yun-sync-repo`), charting (`yun-chart-course`), planning (`yun-grill-plan` → `yun-write-spec` →
`yun-slice-plan`), build (`yun-build-plan` orchestrating, `yun-implement`, `yun-tdd`), knowledge
(`yun-research`, `yun-model-domain`), prototyping (`yun-build-prototype`), isolation
(`yun-spawn-worktree`), adversarial review (`yun-review-work`), handoff (`yun-write-handoff`), and
authoring (`yun-write-skill`). Contracts: `memory-`, `artifact-` (map/spec/ticket kinds, the plan
at rest), `domain-`, `flow-convention` (walking a charted map — the plan in motion), and
`execution-convention` (running one ticket, isolated, to done — the run of one).

The planning-to-build chain now closes end to end: charted → specced → sliced → built, with the
walk cutting each run from the branch the last one produced and leaving that final branch for a
human to review and merge.

What comes next is listed in `skills/README.md` under **Next stage**.

The harness brain is `AGENTS.md`; the skill index is `skills/README.md`; research notes
live in `docs/research/`.
