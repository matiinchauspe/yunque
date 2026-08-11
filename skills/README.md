# Skills — The harness

This is where the workspace harness lives. These skills are **always active** when you
open Claude Code from the workspace root, and they rule over any repo you bring in.

## Naming convention

Every harness skill follows `aw-<verb>-<noun>` (kebab-case). The `aw-` prefix marks it
as part of the workspace harness. See `CLAUDE.md` for the full rule.

## Skills

- `aw-sync-repo/` — brings a repo into the workspace (clones into `repos/<name>/`).
- `aw-spawn-worktree/` — spawns an ephemeral git worktree for a synced repo on its own branch
  (`.worktrees/<repo>/<task>/`), so parallel agents never collide. The light isolation rung —
  creates only; the caller tears it down.
- `aw-write-skill/` — the quality bar for authoring harness skills (invocation, information
  hierarchy, pruning, failure modes). Adapted from Matt Pocock's `writing-great-skills`.
- `aw-grill-plan/` — relentless one-question-at-a-time interview that front-loads the open
  decisions into an airtight spec, recalling prior decisions before asking. Adapted from Matt
  Pocock's `grilling`.
- `aw-write-spec/` — synthesizes a large task's settled decisions into one durable spec
  document (no interview), published through the artifact convention. Adapted from Matt
  Pocock's `to-spec`.
- `aw-slice-plan/` — breaks an approved plan or spec into tracer-bullet vertical slices —
  tickets that each cut a complete path through every layer and declare their blocking edges,
  published through the artifact convention. Adapted from Matt Pocock's `to-tickets`.
- `aw-implement/` — the execution tail: builds a small task in code, or the last step of the
  heavy flow, test-first at the agreed seams, then review and commit. A magnitude guard routes
  large or unsettled work back to planning. Adapted from Matt Pocock's `implement`.
- `aw-tdd/` — the red → green loop: a failing test at a pre-agreed seam, then only enough
  code to pass it, refactor once green. Adapted from Matt Pocock's `tdd`.
- `aw-research/` — delegates reading legwork to a background agent; leaves a cited Markdown
  file in the repo and persists a digest. Adapted from Matt Pocock's `research`.
- `aw-model-domain/` — builds and sharpens a project's domain model: a ubiquitous-language
  glossary (`CONTEXT.md`) and one-paragraph ADRs, captured through the domain convention.
  Adapted from Matt Pocock's `domain-modeling`.
- `aw-write-handoff/` — compacts a session into a disposable baton for the next agent:
  labelled pointers into durable memory plus the live delta. Adapted from Matt Pocock's `handoff`.
- `aw-review-work/` — runs two blind independent reviewers over a decision, plan, spec,
  design, skill, or code change, synthesizes their verdict and, on the user's go-ahead, fixes
  and re-reviews to convergence. Distilled from the global `judgment-day` skill.

## Shared contracts

- `_shared/memory-convention.md` — how any skill remembers or recalls without naming a
  memory tool: declare intent, degrade from an available capability → per-project files
  under `.aw/memory/<project>/` → skip.
- `_shared/artifact-convention.md` — how any skill records tracked work in three kinds (the map
  an effort is charted onto, the spec a plan is written into, the tickets it is sliced into)
  without naming a tracker: declare intent, degrade from an available capability → per-project
  files under `.aw/artifacts/<project>/<feature>/` (the floor — publishing never skips).
- `_shared/domain-convention.md` — how any skill captures or consults a project's domain
  model (glossary + ADRs) without naming a tool: committed in the target repo, with an
  `.aw/domain/<project>/` fallback for repo-less work; capture never skips, consult degrades
  silently.
- `_shared/flow-convention.md` — how any skill walks a charted map without naming a tracker:
  the frontier (takeable tickets), claim/release (concurrency), and the walk rules (cadence,
  autonomy, research auto-dispatch, push-right, empty-frontier). The plan in motion, over the store
  `artifact-convention` holds at rest; native tracker → single-writer file floor.
- `_shared/execution-convention.md` — how any skill runs one ticket to completion without naming a
  runtime: isolate (always a named branch, degrading sandbox → worktree → single-writer inline
  floor) and converge (a bounded loop to the strongest done-signal the ticket offers, else
  fail-fast). The run of one — selection-agnostic, never counts (flow's) nor folds the branch back
  (a serial concern beyond it). Grounded in Matt Pocock's `sandcastle`.

## Next stage

- A dispatching skill that composes flow + artifact + execution — walks the frontier and hands each
  fit ticket to a run (the orchestrator layer; `aw-implement` is the natural home).
- `aw-clean-worktree/` — the teardown half of `aw-spawn-worktree`.
