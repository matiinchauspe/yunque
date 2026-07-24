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
| `aw-write-skill` | Author or edit a skill — the quality bar | auto |
| `aw-grill-plan` | Stress-test a plan/decision into an airtight spec | auto |
| `aw-write-handoff` | Compact a session into a baton for the next agent | user |

*Invocation* maps across tools: **auto** = model-invoked (Claude) / agent-requested (Cursor);
**user** = typed by name (Claude) / slash-command menu (Cursor).

## Structure

```
agentic-workspace/
├─ AGENTS.md                 ← this file: the harness brain (cross-tool)
├─ CLAUDE.md                 → symlink to AGENTS.md (Claude Code reads this)
├─ README.md
├─ .gitignore                ← ignores /repos and /.worktrees
├─ skills/aw-*/SKILL.md      ← THE HARNESS. Canonical. Versioned. Always active.
├─ .claude/skills            → symlink to ../skills  (Claude Code discovery)
├─ .cursor/skills            → symlink to ../skills  (Cursor discovery)
├─ repos/                    ← cloned projects (gitignored)
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

Only the harness: `skills/`, `AGENTS.md`, the `CLAUDE.md` symlink, the `.claude/skills`
and `.cursor/skills` symlinks, `README.md`, `.gitignore`, `docs/`.
The projects' code (`repos/`) and the worktrees (`.worktrees/`) are NOT versioned.
