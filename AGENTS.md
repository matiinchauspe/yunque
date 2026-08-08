# Agentic Workspace

An umbrella workspace to work on any repo under your own harness, decoupled from any
single project. Projects are interchangeable; the harness rules. Tool-agnostic: the same
harness drives **Claude Code** and **Cursor** (both consume the open `SKILL.md` standard).

## Workspace rules

1. **ALWAYS open your agent from the workspace root.**
   Don't enter a repo and open your agent there — *bring the repo into the workspace*.
   That's the only way the `skills/` harness stays loaded and rules. If you open from
   `repos/<repo>/`, the agent loads that project's skills and NOT the harness.
   You don't go into the repo: you bring the repo into the workspace.

2. **Repos enter through the `aw-sync-repo` skill.**
   Cloned into `repos/<name>/` (gitignored), each with its own `.git`.

3. **Parallelism goes through the `aw-spawn-worktree` skill.**
   Creates ephemeral worktrees in `.worktrees/<repo>/<task>/`. NEVER inside the repo
   (avoids recursive scanning and one agent editing another's branch).

4. **Worktree cleanup:** run `git worktree prune` BEFORE deleting the source repo.
   Git stores absolute paths in the gitdir; if you move the workspace, use
   `git worktree repair`.

## Skills

Harness skills live **once** in `skills/aw-*/SKILL.md` (canonical). Both agents discover
them through relative symlinks — no duplication, no per-tool translation:

- `.claude/skills` → `../skills`  (Claude Code discovery)
- `.cursor/skills` → `../skills`  (Cursor discovery)

Index of the current harness (name — when to reach for it):

| Skill | When | Invocation |
| ----- | ---- | ---------- |
| `aw-sync-repo` | Bring an external repo into the workspace | user |
| `aw-spawn-worktree` | Spawn an isolated worktree so parallel work never collides | auto |
| `aw-write-skill` | Author or edit a skill — the quality bar | auto |
| `aw-chart-course` | Chart work too big for one session as a live map of decision tickets, cleared one at a time | user |
| `aw-grill-plan` | Stress-test a plan/decision into airtight, settled decisions | auto |
| `aw-write-spec` | Synthesize a settled conversation into one durable spec document | auto |
| `aw-slice-plan` | Break an approved plan/spec into tracer-bullet tickets | auto |
| `aw-implement` | When the next step is writing code — from a spec, tickets, or a described task | auto |
| `aw-tdd` | When building test-first — the red → green loop at a pre-agreed seam | auto |
| `aw-research` | Delegate reading legwork; get back a cited Markdown file | auto |
| `aw-build-prototype` | Answer a design question by building it — a hand-driven state model, or rival UI variants | auto |
| `aw-model-domain` | Pin down a project's ubiquitous language and record the ADRs behind its shape | auto |
| `aw-write-handoff` | Compact a session into a baton for the next agent | user |
| `aw-review-work` | Review a decision/plan/spec/design/skill/code with two blind adversarial reviewers | auto |

*Invocation* maps across tools: **auto** = model-invoked (Claude) / agent-requested (Cursor);
**user** = typed by name (Claude) / slash-command menu (Cursor).

**Adversarial review during coexistence.** While the global `judgment-day` skill is still
installed, it owns the judge phrasing — `judgment day`, `judgment-day`, `review adversarial`,
`dual review`, `doble review`, `juzgar`, `juzgá`, `que lo juzguen` — route all of it to the
global. `aw-review-work` fires via its `auto`/offer path and its own review phrases
(`review this work`, `revisá esto`, `revisá este trabajo`), which keeps it clear of the
global's judge phrases. When the global is retired, its phrases move to `aw-review-work`.

Skills lean on shared, tool-agnostic contracts under `skills/_shared/`. A skill declares
intent and never binds a specific tool; each contract resolves that intent to a concrete home
and degrades by its own rules, spelled out below:

- `memory-convention.md` — remember/recall across sessions (persist/recall) →
  `.aw/memory/<project>/` → skip.
- `artifact-convention.md` — record tracked work in three kinds: the map an effort is charted
  onto, the spec a plan is written into, and the tickets it breaks into (publish/fetch; list
  the decision tickets) →
  `.aw/artifacts/<project>/<feature>/` (`map.md` + `spec.md` + decision tickets under
  `decisions/` + numbered implementation ticket files; the floor — publishing is a deliverable,
  so it never skips).
- `domain-convention.md` — capture/consult a project's domain model: its ubiquitous-language
  glossary and the ADRs behind its shape (capture/consult) → committed in the target repo
  (`CONTEXT.md` + `docs/adr/`) → `.aw/domain/<project>/` (same layout) when there is no repo. It
  is a project deliverable, so capture never skips; consulting an absent model proceeds silently.
- `flow-convention.md` — walk a charted map: which ticket is takeable now, who holds it, at what
  pace, what runs unattended (frontier/claim/release + cadence, autonomy, auto-dispatch,
  push-right, empty-frontier).
  The plan in motion, where `artifact-convention` holds it at rest — it computes the frontier over
  that store and keeps none of its own; native tracker → file floor (single-writer, so the walk
  degrades to serial and claiming goes mute).

## Structure

```
agentic-workspace/
├─ AGENTS.md                 ← this file: the harness brain (cross-tool)
├─ CLAUDE.md                 → symlink to AGENTS.md (Claude Code reads this)
├─ README.md
├─ .gitignore                ← ignores /repos, /.worktrees and /.aw
├─ skills/aw-*/SKILL.md      ← THE HARNESS. Canonical. Versioned. Always active.
├─ skills/_shared/           ← cross-skill contracts (memory-, artifact-, domain-, flow-convention)
├─ .claude/skills            → symlink to ../skills  (Claude Code discovery)
├─ .cursor/skills            → symlink to ../skills  (Cursor discovery)
├─ repos/                    ← cloned projects (gitignored)
├─ .worktrees/<repo>/<task>/ ← ephemeral worktrees (gitignored, on-demand)
├─ .aw/memory/<project>/     ← per-project memory file fallback (gitignored)
├─ .aw/artifacts/<project>/<feature>/ ← per-project ticket file fallback (gitignored)
└─ .aw/domain/<project>/     ← domain-model fallback when work isn't tied to a repo (gitignored)
```

The domain model's real home is the target repo itself (`repos/<name>/CONTEXT.md` + `docs/adr/`),
committed with the code; `.aw/domain/` only catches work not tied to a repo.

## Skill naming convention

Every harness skill follows: **`aw-<verb>-<noun>`** — kebab-case, lowercase, and the
directory name must match the skill name (Agent Skills spec).

- `aw-` prefix → the skill belongs to the workspace harness (distinguishes it from
  a repo's own skills or global skills in autocomplete).
- After the prefix, **verb-noun** (imperative — reads like a command).
- **Leading-word exception:** a skill whose subject is a strong pretrained term may keep that
  term in place of strict verb-noun, when splitting it would cost invocation reliability —
  e.g. `aw-tdd` (the `TDD` leading word) or `aw-research`. Reach for this rarely; the default
  is verb-noun.
- Examples: `aw-sync-repo`, `aw-spawn-worktree`, `aw-clean-worktree`, `aw-list-repos`.

## What this repo versions

Only the harness: `skills/` (including `skills/_shared/`), `AGENTS.md`, the `CLAUDE.md`
symlink, the `.claude/skills` and `.cursor/skills` symlinks, `README.md`, `.gitignore`, `docs/`.
The projects' code (`repos/`), the worktrees (`.worktrees/`), and the per-project memory,
artifact, and domain-model fallback files (`.aw/`) are NOT versioned.
