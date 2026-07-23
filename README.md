# Agentic Workspace

A **project-agnostic workspace**: an umbrella directory, with its own repo, that hosts
your agent harness (skills, rules, flows) decoupled from any single project. You bring
whatever repo you want inside and work it with YOUR harness — not the other way around.

## Core idea

The harness rules. Projects are interchangeable work material that comes and goes.
This is dependency inversion: the project depends on the workspace, never the workspace
on a project.

## How to use it (intended flow)

1. Open Claude Code from the **root** of this workspace → your harness loads.
2. `sync-repo <url|name>` → clones the project into `repos/<name>/`.
3. Work. If you need to parallelize → `spawn-worktree <repo> <task>` → isolated agent
   in `.worktrees/<repo>/<task>/`.
4. When done → merge, cleanup (`git worktree prune`), the worktree disappears.

## Status

- **Stage 1 (current):** foundations — structure, rules, `.gitignore`, written design.
- **Next stages:** `sync-repo` and `spawn-worktree` skills, and the rest of the harness.

See the full design in `docs/superpowers/specs/`.
