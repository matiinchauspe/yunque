# Skills — The harness

This is where the workspace harness lives. These skills are **always active** when you
open Claude Code from the workspace root, and they rule over any repo you bring in.

## Next stage

- `sync-repo/` — brings a repo into the workspace (clones into `repos/<name>/`).
- `spawn-worktree/` — creates ephemeral worktrees for parallel agents.
