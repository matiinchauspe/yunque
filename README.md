<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/yunque-dark.svg">
  <img src="assets/yunque-light.svg" alt="Yunque" width="200">
</picture>

# Yunque

**The harness rules. Projects are interchangeable work material.**

[![License: MIT](https://img.shields.io/badge/license-MIT-D97706?style=flat-square)](LICENSE)
![Skills](https://img.shields.io/badge/skills-18-1F2328?style=flat-square)
![Agents](https://img.shields.io/badge/agents-Claude%20Code%20%C2%B7%20Cursor-1F2328?style=flat-square)

</div>

A **project-agnostic workspace**: an umbrella directory, with its own repo, that hosts
your agent harness (skills, rules, flows) decoupled from any single project. You bring
whatever repo you want inside and work it with YOUR harness — not the other way around.

## Core idea

The harness rules. Projects are interchangeable work material that comes and goes.
This is dependency inversion: the project depends on the workspace, never the workspace
on a project.

## Install

Yunque is not a package you add to a project. It's a workspace you clone, and open your
agent *inside of*.

```bash
git clone https://github.com/matiinchauspe/yunque.git
cd yunque
```

That's the whole install. The skill-discovery paths are **committed relative symlinks**, so a
clone reproduces a working harness anywhere on disk — nothing to build, register or configure:

| Path | points to | read by |
| ---- | --------- | ------- |
| `.claude/skills` | `../skills` | Claude Code |
| `.cursor/skills` | `../skills` | Cursor |
| `CLAUDE.md` | `AGENTS.md` | Claude Code |

**On Windows**, git materializes symlinks only with `core.symlinks=true` (needs Developer Mode
or an elevated shell). Without it you get plain text files containing the target path, and no
skill loads at all. Clone with `git clone -c core.symlinks=true …`.

Then the rule the whole thing rests on:

> **Open your agent from the root of this workspace** — never from inside a repo you're working on.

Open it in `repos/<project>/` and the agent loads *that project's* skills; the harness never
loads. You don't go into the repo — you bring the repo into the workspace.

To verify it took, your agent should list 18 skills prefixed `yun-`.

If you will edit the harness itself, turn on its pre-commit hook, which runs the `lenses/` suite
over the staged tree:

```bash
git config core.hooksPath .githooks
```

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
5. **Hand it over.** `/yun-write-pr` writes the PR body for the branch — its shape, the evidence
   it works, how dangerous it is to merge. Ending a session mid-work → `/yun-write-handoff`
   leaves a baton for the next one.

Along the way: a bug that resisted the obvious fix → `yun-diagnose-bug`; an explanation that did
not land → `/yun-show-me`.

### Skills you type

Six skills run only when you type them; the other twelve fire on their own when the work
calls for them.

| Skill | When |
| ----- | ---- |
| `/yun-sync-repo` | Bring an external repo into the workspace |
| `/yun-chart-course` | Chart work too big for one session as a map of decisions |
| `/yun-build-plan` | Build a sliced plan to completion, leaving one branch to review |
| `/yun-write-handoff` | Compact a session into a baton for the next agent |
| `/yun-show-me` | Re-pitch what did not land, with the smallest view that shows it |
| `/yun-write-pr` | Write a PR body that shows the change |

## Current state

The harness is operational — 18 skills over six shared contracts: bring-in
(`yun-sync-repo`), charting (`yun-chart-course`), planning (`yun-grill-plan` → `yun-write-spec` →
`yun-slice-plan`), build (`yun-build-plan` orchestrating, `yun-implement`, `yun-tdd`), repair
(`yun-diagnose-bug`), knowledge (`yun-research`, `yun-model-domain`), prototyping
(`yun-build-prototype`), isolation (`yun-spawn-worktree`), adversarial review (`yun-review-work`),
explanation (`yun-show-me`), PR writing (`yun-write-pr`), handoff (`yun-write-handoff`), and
authoring (`yun-write-skill`). Contracts: `memory-`, `artifact-`
(map/spec/ticket kinds, the plan at rest), `domain-`, `flow-convention` (walking a charted map —
the plan in motion), `execution-convention` (running one ticket, isolated, to done — the run of
one), and `practice-convention` (writing code the way the target repo already writes it).

The planning-to-build chain now closes end to end: charted → specced → sliced → built, with the
walk cutting each run from the branch the last one produced and leaving that final branch for a
human to review and merge.

What comes next is listed in `skills/README.md` under **Next stage**.

The harness brain is `AGENTS.md`; the skill index is `skills/README.md`.

## Credits

The skills here are original writing, but several of them adapt patterns worked out in public by
other people. The lineage is recorded per skill in [`skills/README.md`](skills/README.md) — each
adapted skill names the work it learned from.

- **[mattpocock/skills](https://github.com/mattpocock/skills)** (MIT) — the skill-authoring bar,
  and the patterns behind charting, grilling, spec-writing, slicing, implementing, TDD, diagnosing, research,
  prototyping, domain modeling, handoff, re-pitching and PR writing.
- **[humanlayer/skills](https://github.com/humanlayer/skills)** (MIT) — Dex Horthy's `show-me`,
  the visual vocabulary behind `yun-show-me` and `yun-write-pr`, and his `visual-pr`, behind the
  PR body's links, its one-sentence why and the contract view.
- **[mattpocock/sandcastle](https://github.com/mattpocock/sandcastle)** (MIT) — the frontier walk
  and isolated per-ticket execution behind `yun-build-plan` and `execution-convention`.

## License

MIT — see [LICENSE](LICENSE). Copyright (c) 2026 Matias Inchauspe.
