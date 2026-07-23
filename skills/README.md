# Skills — The harness

This is where the workspace harness lives. These skills are **always active** when you
open Claude Code from the workspace root, and they rule over any repo you bring in.

## Naming convention

Every harness skill follows `ws-<verb>-<noun>` (kebab-case). The `ws-` prefix marks it
as part of the workspace harness. See `CLAUDE.md` for the full rule.

## Next stage

- `ws-sync-repo/` — brings a repo into the workspace (clones into `repos/<name>/`).
- `ws-spawn-worktree/` — creates ephemeral worktrees for parallel agents.
