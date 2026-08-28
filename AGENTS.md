# Yunque

An umbrella workspace to work on any repo under your own harness, decoupled from any
single project. Projects are interchangeable; the harness rules. Tool-agnostic: the same
harness drives **Claude Code** and **Cursor** (both consume the open `SKILL.md` standard).

## Workspace rules

1. **ALWAYS open your agent from the workspace root.**
   Don't enter a repo and open your agent there — *bring the repo into the workspace*.
   That's the only way the `skills/` harness stays loaded and rules. If you open from
   `repos/<repo>/`, the agent loads that project's skills and NOT the harness.
   You don't go into the repo: you bring the repo into the workspace.

2. **Repos enter through the `yun-sync-repo` skill.**
   Cloned into `repos/<name>/` (gitignored), each with its own `.git`.

3. **Parallelism goes through the `yun-spawn-worktree` skill.**
   Creates ephemeral worktrees in `.worktrees/<repo>/<task>/`. NEVER inside the repo
   (avoids recursive scanning and one agent editing another's branch).

4. **Worktree cleanup:** run `git worktree prune` BEFORE deleting the source repo.
   Git stores absolute paths in the gitdir; if you move the workspace, use
   `git worktree repair`.

## Skills

Harness skills live **once** in `skills/yun-*/SKILL.md` (canonical). Both agents discover
them through relative symlinks — no duplication, no per-tool translation:

- `.claude/skills` → `../skills`  (Claude Code discovery)
- `.cursor/skills` → `../skills`  (Cursor discovery)

Index of the current harness (name — when to reach for it):

| Skill | When | Invocation |
| ----- | ---- | ---------- |
| `yun-sync-repo` | Bring an external repo into the workspace | user |
| `yun-spawn-worktree` | Spawn an isolated worktree so parallel work never collides | auto |
| `yun-write-skill` | Author or edit a skill — the quality bar | auto |
| `yun-chart-course` | Chart work too big for one session as a live map of decision tickets, cleared one at a time | user |
| `yun-grill-plan` | Stress-test a plan/decision into airtight, settled decisions | auto |
| `yun-write-spec` | Synthesize a settled conversation into one durable spec document | auto |
| `yun-slice-plan` | Break an approved plan/spec into tracer-bullet tickets | auto |
| `yun-build-plan` | Build a sliced plan to completion, leaving one branch to review | user |
| `yun-implement` | When the next step is writing code — from a spec, tickets, or a described task | auto |
| `yun-tdd` | When building test-first — the red → green loop at a pre-agreed seam | auto |
| `yun-research` | Delegate reading legwork; get back a cited Markdown file | auto |
| `yun-build-prototype` | Answer a design question by building it — a hand-driven state model, or rival UI variants | auto |
| `yun-model-domain` | Pin down a project's ubiquitous language and record the ADRs behind its shape | auto |
| `yun-write-handoff` | Compact a session into a baton for the next agent | user |
| `yun-review-work` | Review a decision/plan/spec/design/skill/code with two blind adversarial reviewers | auto |

*Invocation* maps across tools: **auto** = model-invoked (Claude) / agent-requested (Cursor);
**user** = typed by name (Claude) / slash-command menu (Cursor).

**Adversarial review during coexistence.** While the global `judgment-day` skill is still
installed, it owns the judge phrasing — `judgment day`, `judgment-day`, `review adversarial`,
`dual review`, `doble review`, `juzgar`, `juzgá`, `que lo juzguen` — route all of it to the
global. `yun-review-work` fires via its `auto`/offer path and its own review phrases
(`review this work`, `revisá esto`, `revisá este trabajo`), which keeps it clear of the
global's judge phrases. When the global is retired, its phrases move to `yun-review-work`.

Skills lean on shared, tool-agnostic contracts under `skills/_shared/`. A skill declares
intent and never binds a specific tool; each contract resolves that intent to a concrete home
and degrades by its own rules, spelled out below:

- `memory-convention.md` — remember/recall across sessions (persist/recall) →
  `.yun/memory/<project>/` → skip.
- `artifact-convention.md` — record tracked work in three kinds: the map an effort is charted
  onto, the spec a plan is written into, and the tickets it breaks into (publish/fetch/resolve;
  list one ticket **namespace** — decision or implementation, never both, since they are separate
  sets on independent counters, joined by the ticket **number**) →
  `.yun/artifacts/<project>/<feature>/` (`map.md` + `spec.md` + decision tickets under
  `decisions/` + numbered implementation ticket files; the floor — publishing is a deliverable,
  so it never skips).
- `domain-convention.md` — capture/consult a project's domain model: its ubiquitous-language
  glossary and the ADRs behind its shape (capture/consult) → committed in the target repo
  (`CONTEXT.md` + `docs/adr/`) → `.yun/domain/<project>/` (same layout) when there is no repo. It
  is a project deliverable, so capture never skips; consulting an absent model proceeds silently.
- `flow-convention.md` — walk a charted map: which ticket is takeable now, who holds it, at what
  pace, what runs unattended (frontier/claim/release + cadence, autonomy, auto-dispatch,
  push-right, empty-frontier).
  The plan in motion, where `artifact-convention` holds it at rest — it computes the frontier over
  that store and keeps none of its own; native tracker → file floor (single-writer, so the walk
  degrades to serial and claiming goes mute). Scoped to the **decision** walk: its walk rules key
  off a `Type:` implementation tickets do not carry, so `yun-build-plan` computes its own frontier.
- `execution-convention.md` — run one ticket to completion, isolated: the workspace it runs in and
  the loop that drives it to done (isolate/converge). Isolation degrades sandbox → worktree
  (`yun-spawn-worktree`) → single-writer inline floor, cutting the branch from an optional `base`
  the caller passes when it holds one (else HEAD); tier 1 is any sandbox — the agent's or one the
  target repo declares — qualifying only by SHAPE, a per-ticket entry point (a whole orchestrator
  answers a different verb). Converge is a bounded loop ending on the strongest done-signal the
  ticket offers, else fail-fast. The run of one — it never counts (flow's) nor integrates the branch
  it leaves (`yun-build-plan`'s, which does that by cutting each run from the last one's branch);
  selection-agnostic, a dispatching skill hands it the ticket.

## Structure

```
yunque/
├─ AGENTS.md                 ← this file: the harness brain (cross-tool)
├─ CLAUDE.md                 → symlink to AGENTS.md (Claude Code reads this)
├─ README.md
├─ LICENSE                   ← MIT
├─ assets/                   ← the Yunque mark (light/dark SVG for the README)
├─ .gitignore                ← ignores /repos, /.worktrees and /.yun
├─ skills/yun-*/SKILL.md      ← THE HARNESS. Canonical. Versioned. Always active.
├─ skills/_shared/           ← cross-skill contracts (memory-, artifact-, domain-, flow-, execution-convention)
├─ .claude/skills            → symlink to ../skills  (Claude Code discovery)
├─ .cursor/skills            → symlink to ../skills  (Cursor discovery)
├─ repos/                    ← cloned projects (gitignored)
├─ .worktrees/<repo>/<task>/ ← ephemeral worktrees (gitignored, on-demand)
├─ .yun/memory/<project>/     ← per-project memory file fallback (gitignored)
├─ .yun/artifacts/<project>/<feature>/ ← per-project ticket file fallback (gitignored)
└─ .yun/domain/<project>/     ← domain-model fallback when work isn't tied to a repo (gitignored)
```

The domain model's real home is the target repo itself (`repos/<name>/CONTEXT.md` + `docs/adr/`),
committed with the code; `.yun/domain/` only catches work not tied to a repo.

## Skill naming convention

Every harness skill follows: **`yun-<verb>-<noun>`** — kebab-case, lowercase, and the
directory name must match the skill name (Agent Skills spec).

- `yun-` prefix → the skill belongs to the workspace harness (distinguishes it from
  a repo's own skills or global skills in autocomplete).
- After the prefix, **verb-noun** (imperative — reads like a command).
- **Leading-word exception:** a skill whose subject is a strong pretrained term may keep that
  term in place of strict verb-noun, when splitting it would cost invocation reliability —
  e.g. `yun-tdd` (the `TDD` leading word) or `yun-research`. Reach for this rarely; the default
  is verb-noun.
- Examples: `yun-sync-repo`, `yun-spawn-worktree`, `yun-clean-worktree`, `yun-list-repos`.

## What this repo versions

Only the harness: `skills/` (including `skills/_shared/`), `AGENTS.md`, the `CLAUDE.md`
symlink, the `.claude/skills` and `.cursor/skills` symlinks, `README.md`, `LICENSE`,
`assets/` and `.gitignore`. Research notes under `docs/` stay local and are gitignored.
The projects' code (`repos/`), the worktrees (`.worktrees/`), and the per-project memory,
artifact, and domain-model fallback files (`.yun/`) are NOT versioned.
