# Agentic Workspace

An umbrella workspace to work on any repo under your own harness, decoupled from any
single project. Projects are interchangeable; the harness rules.

## Workspace rules

1. **ALWAYS open Claude Code from the workspace root.**
   Don't enter a repo and open Claude there — *bring the repo into the workspace*.
   That's the only way the `skills/` harness stays loaded and rules. If you open from
   `repos/<repo>/`, Claude loads that project's skills and NOT the harness.
   You don't go into the repo: you bring the repo into the workspace.

2. **Repos enter through the `aw-sync-repo` skill.**
   Cloned into `repos/<name>/` (gitignored), each with its own `.git`.

3. **Parallelism goes through the `aw-spawn-worktree` skill.**
   Creates ephemeral worktrees in `.worktrees/<repo>/<task>/`. NEVER inside the repo
   (avoids recursive scanning and one agent editing another's branch).

4. **Worktree cleanup:** run `git worktree prune` BEFORE deleting the source repo.
   Git stores absolute paths in the gitdir; if you move the workspace, use
   `git worktree repair`.

## Structure

```
agentic-workspace/
├─ CLAUDE.md                 ← this file (workspace rules)
├─ README.md
├─ .gitignore               ← ignores /repos and /.worktrees
├─ skills/                  ← THE HARNESS. Versioned. Always active.
├─ repos/                   ← cloned projects (gitignored)
└─ .worktrees/<repo>/<task>/ ← ephemeral worktrees (gitignored, on-demand)
```

## Skill naming convention

Every harness skill follows: **`aw-<verb>-<noun>`** — kebab-case, lowercase, and the
directory name must match the skill name (Agent Skills spec).

- `aw-` prefix → the skill belongs to the workspace harness (distinguishes it from
  a repo's own skills or global skills in autocomplete).
- After the prefix, **verb-noun** (imperative — reads like a command).
- Examples: `aw-sync-repo`, `aw-spawn-worktree`, `aw-clean-worktree`, `aw-list-repos`.

## What this repo versions

Only the harness: `skills/`, `CLAUDE.md`, `README.md`, `.gitignore`, `docs/`.
The projects' code (`repos/`) and the worktrees (`.worktrees/`) are NOT versioned.
