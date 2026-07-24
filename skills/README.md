# Skills — The harness

This is where the workspace harness lives. These skills are **always active** when you
open Claude Code from the workspace root, and they rule over any repo you bring in.

## Naming convention

Every harness skill follows `aw-<verb>-<noun>` (kebab-case). The `aw-` prefix marks it
as part of the workspace harness. See `CLAUDE.md` for the full rule.

## Skills

- `aw-sync-repo/` — brings a repo into the workspace (clones into `repos/<name>/`).
- `aw-write-skill/` — the quality bar for authoring harness skills (invocation, information
  hierarchy, pruning, failure modes). Adapted from Matt Pocock's `writing-great-skills`.
- `aw-grill-plan/` — relentless one-question-at-a-time interview that front-loads the open
  decisions into an airtight spec, checking engram before asking. Adapted from Matt Pocock's
  `grilling`.

## Next stage

- `aw-spawn-worktree/` — creates ephemeral worktrees for parallel agents.
