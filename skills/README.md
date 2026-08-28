# Skills — The harness

This is where the workspace harness lives. These skills are **always active** when you
open Claude Code from the workspace root, and they rule over any repo you bring in.

## Naming convention

Every harness skill follows `yun-<verb>-<noun>` (kebab-case). The `yun-` prefix marks it
as part of the workspace harness. See `CLAUDE.md` for the full rule.

## Skills

- `yun-sync-repo/` — brings a repo into the workspace (clones into `repos/<name>/`).
- `yun-spawn-worktree/` — spawns an ephemeral git worktree for a synced repo on its own branch
  (`.worktrees/<repo>/<task>/`), so parallel agents never collide. The light isolation rung —
  creates only; the caller tears it down.
- `yun-write-skill/` — the quality bar for authoring harness skills (invocation, information
  hierarchy, pruning, failure modes). Adapted from Matt Pocock's `writing-great-skills`.
- `yun-chart-course/` — charts an effort too big for one session as a live map of decision
  tickets, cleared one at a time until the way to the destination is clear. Adapted from Matt
  Pocock's `wayfinder`.
- `yun-grill-plan/` — relentless one-question-at-a-time interview that front-loads the open
  decisions into an airtight spec, recalling prior decisions before asking. Adapted from Matt
  Pocock's `grilling`.
- `yun-write-spec/` — synthesizes a large task's settled decisions into one durable spec
  document (no interview), published through the artifact convention. Adapted from Matt
  Pocock's `to-spec`.
- `yun-slice-plan/` — breaks an approved plan or spec into tracer-bullet vertical slices —
  tickets that each cut a complete path through every layer and declare their blocking edges,
  published through the artifact convention. Adapted from Matt Pocock's `to-tickets`.
- `yun-build-plan/` — the orchestrator: walks a sliced plan's implementation frontier, dispatches
  each ticket to an isolated run behind a whole-tree gate, and leaves ONE branch for a human to
  review. Serial, so it cuts each run from the branch the last one produced: the chain integrates
  by construction, the final branch already holds every ticket, and **the walk itself never writes
  to git** — every git command it runs is a read. It also **keeps no state of its own**: each pass
  re-derives the chain's tip and its skip set from the `aw/<slug>/` branch namespace and the ticket
  statuses, so every pass is a resume, and deleting a failed run's branch is how a human retries it.
  `LIMITS.md` declares what the mechanism does not guarantee. Adapted from Matt Pocock's
  `parallel-planner` sandcastle template, minus its merge phase, which only exists because that
  template fans out — its frontier counts file overlap as a blocking edge where ours counts only
  declared dependencies.
- `yun-implement/` — the execution tail: builds a small task in code, or the last step of the
  heavy flow, test-first at the agreed seams, then review and commit. A magnitude guard routes
  large or unsettled work back to planning. Adapted from Matt Pocock's `implement`.
- `yun-tdd/` — the red → green loop: a failing test at a pre-agreed seam, then only enough
  code to pass it, refactor once green. Adapted from Matt Pocock's `tdd`.
- `yun-research/` — delegates reading legwork to a background agent; leaves a cited Markdown
  file in the repo and persists a digest. Adapted from Matt Pocock's `research`.
- `yun-build-prototype/` — answers a design question by building it: a hand-driven state model,
  or rival UI variants side by side, thrown away once it has ruled. Adapted from Matt Pocock's
  `prototype`.
- `yun-model-domain/` — builds and sharpens a project's domain model: a ubiquitous-language
  glossary (`CONTEXT.md`) and one-paragraph ADRs, captured through the domain convention.
  Adapted from Matt Pocock's `domain-modeling`.
- `yun-write-handoff/` — compacts a session into a disposable baton for the next agent:
  labelled pointers into durable memory plus the live delta. Adapted from Matt Pocock's `handoff`.
- `yun-review-work/` — runs two blind independent reviewers over a decision, plan, spec,
  design, skill, or code change, synthesizes their verdict and, on the user's go-ahead, fixes
  and re-reviews to convergence. Distilled from the global `judgment-day` skill.

## Shared contracts

- `_shared/memory-convention.md` — how any skill remembers or recalls without naming a
  memory tool: declare intent, degrade from an available capability → per-project files
  under `.yun/memory/<project>/` → skip.
- `_shared/artifact-convention.md` — how any skill records tracked work in three kinds (the map
  an effort is charted onto, the spec a plan is written into, the tickets it is sliced into)
  without naming a tracker: declare intent, degrade from an available capability → per-project
  files under `.yun/artifacts/<project>/<feature>/` (the floor — publishing never skips), and
  `resolve` moves one ticket to a terminal state. `list` takes a ticket **namespace** — decision
  or implementation — since the two are separate sets on independent counters, and the ticket
  **number** is the join key its blocking edges name.
- `_shared/domain-convention.md` — how any skill captures or consults a project's domain
  model (glossary + ADRs) without naming a tool: committed in the target repo, with an
  `.yun/domain/<project>/` fallback for repo-less work; capture never skips, consult degrades
  silently.
- `_shared/flow-convention.md` — how any skill walks a charted map without naming a tracker:
  the frontier (takeable tickets), claim/release (concurrency), and the walk rules (cadence,
  autonomy, research auto-dispatch, push-right, empty-frontier). The plan in motion, over the store
  `artifact-convention` holds at rest; native tracker → single-writer file floor. Scoped to the
  decision walk — `yun-build-plan` computes its own frontier over implementation tickets.
- `_shared/execution-convention.md` — how any skill runs one ticket to completion without naming a
  runtime: isolate (always a named branch, cut from an optional `base` a caller passes when it
  holds one, degrading a per-ticket sandbox → worktree → single-writer inline floor, every rung
  leaving that branch as a ref in the host repo) and converge
  (a bounded loop to the strongest done-signal the ticket offers, else fail-fast). The run of one —
  selection-agnostic, never counts (flow's) nor integrates the branch it leaves (`yun-build-plan`'s,
  which does that by cutting each run from the last one's branch). Grounded in Matt Pocock's
  `sandcastle`.

## Next stage

- An unattended path for `yun-review-work` — bounded to one pass, so a dispatched run can be
  reviewed before the next run is chained onto it instead of only at the end.
- `yun-resolve-conflict/` — a merge-conflict skill. Nothing inside a walk can hit one —
  `yun-build-plan` is serial, so each run is cut from the last and the branches never diverge — but
  the merge *out* of a walk can: the chain is cut from a mainline the walk reads once and never
  refreshes, so the human merging it back meets whatever landed meanwhile. That merge, and a parallel walk, are the two
  cases this would serve.
- `yun-clean-worktree/` — the teardown half of `yun-spawn-worktree`.
