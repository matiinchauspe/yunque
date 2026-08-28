---
name: yun-sync-repo
description: Bring an external repo into the workspace as work material — cloned into repos/<name>/, gitignored, with its own .git.
disable-model-invocation: true
---

# yun-sync-repo

Brings an external repository into the workspace as work material. The repo is cloned
into `repos/<name>/`, which is gitignored — the workspace versions only the harness,
never the projects' code.

## Preconditions

- You MUST be at the workspace root (the directory containing `CLAUDE.md` and `skills/`).
  Verify it: `eza -a | rg -q '^CLAUDE.md$'` and a `skills/` directory both exist.
  If you are not at the root, STOP and tell the user to open Claude Code from the
  workspace root (see CLAUDE.md rule 1). Do not clone from anywhere else.

## Inputs

- A git URL (`https://…/repo.git`, `git@…:owner/repo.git`), OR
- A GitHub shorthand `owner/repo`, OR
- An explicit `<name>` for the target folder (optional; defaults to the repo basename).

## Steps

1. **Resolve the target name.** Use the explicit name if the user gave one; otherwise
   derive it from the URL basename without the `.git` suffix. Keep it kebab-case.

2. **Check for collision.** Look at `repos/<name>/`:
   - If it already exists and is a clone of the same remote → do NOT re-clone. Report it
     is already synced, and offer `git -C repos/<name> fetch --all --prune` to update.
   - If it exists but points to a different remote, or is a dirty non-repo directory →
     STOP and ask the user how to proceed. Never overwrite blindly.

3. **Clone.** `git clone <url> repos/<name>` — a full clone (worktrees need full history;
   do NOT use `--depth`).

4. **Verify the clone.**
   - `git -C repos/<name> rev-parse --is-inside-work-tree` returns `true`.
   - `git -C repos/<name> remote -v` shows the expected remote.

5. **Confirm isolation.** `repos/` is gitignored at the workspace level, so `git status`
   at the workspace root must NOT list the new repo. If it does, the workspace
   `.gitignore` is broken — fix it before continuing.

6. **Report.** Give the ready path `repos/<name>/` and its default branch.

## Notes

- Never clone into the workspace root or into `skills/`. Only into `repos/<name>/`.
- Do NOT `cd` into `repos/<name>/` to keep working — stay at the workspace root so the
  harness stays loaded (CLAUDE.md rule 1). Operate on the repo via `git -C repos/<name>`,
  or spawn an isolated worktree with `yun-spawn-worktree`.
