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
  decisions into an airtight spec, recalling prior decisions before asking. Adapted from Matt
  Pocock's `grilling`.
- `aw-slice-plan/` — breaks an approved plan or spec into tracer-bullet vertical slices —
  tickets that each cut a complete path through every layer and declare their blocking edges,
  published through the artifact convention. Adapted from Matt Pocock's `to-tickets`.
- `aw-research/` — delegates reading legwork to a background agent; leaves a cited Markdown
  file in the repo and persists a digest. Adapted from Matt Pocock's `research`.
- `aw-write-handoff/` — compacts a session into a disposable baton for the next agent:
  labelled pointers into durable memory plus the live delta. Adapted from Matt Pocock's `handoff`.

## Shared contracts

- `_shared/memory-convention.md` — how any skill remembers or recalls without naming a
  memory tool: declare intent, degrade from an available capability → per-project files
  under `.aw/memory/<project>/` → skip.
- `_shared/artifact-convention.md` — how any skill records tracked work (the tickets a plan
  is sliced into) without naming a tracker: declare intent, degrade from an available
  capability → per-project files under `.aw/artifacts/<project>/<feature>/` (the floor —
  publishing never skips).

## Next stage

- `aw-spawn-worktree/` — creates ephemeral worktrees for parallel agents.
