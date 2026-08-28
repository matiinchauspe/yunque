---
name: yun-spawn-worktree
description: Spawn an isolated git worktree for a synced repo so parallel work never collides. Use when the user wants to parallelize, work a task in isolation, or run one of several tickets on its own branch — or when another skill (e.g. yun-implement) needs an isolated worktree to build in. Creates .worktrees/<repo>/<task>/ on its own branch.
---

# yun-spawn-worktree

Spawn an ephemeral **worktree** — a second working tree of a synced repo, on its own branch,
so parallel agents each get an isolated tree and branch instead of editing or scanning each
other's. It only creates the worktree; removing it is the caller's job once the work lands
(`git worktree remove`, or a future `yun-clean-worktree`). This is the light isolation rung —
pure git, local, no sandbox — the seam a heavier execution substrate plugs in above.

## Preconditions

- You MUST be at the workspace root (the dir with `CLAUDE.md` and `skills/`). Stay there —
  operate with `git -C`, never `cd` into the repo or the worktree (CLAUDE.md rule 1).
- `repos/<repo>/` exists and is a FULL clone — worktrees need full history, so a shallow
  clone won't do (`yun-sync-repo` clones full by default).

## Inputs

- `<repo>` — the synced repo under `repos/<repo>/`.
- `<task>` — a kebab-case slug for the work; names both the worktree dir and its branch. It may
  carry slash-separated segments (`<prefix>/<ticket>-<nonce>`) so a caller spawning a whole set
  can group them under one branch namespace it can list and prune later; git takes the slashes in
  a branch name, and the worktree simply nests to match.
- `<base>` — the ref to branch from (optional; defaults to the repo's current `HEAD`).

## Steps

1. **Resolve and check for collision.** Confirm `repos/<repo>/` is a repo. If
   `.worktrees/<repo>/<task>/` already exists, or a branch named `<task>` already exists in
   the repo, STOP and ask — never clobber another agent's worktree or branch. Done when the
   target path and branch name are both free.

2. **Spawn the worktree on a new branch.** Add it at an ABSOLUTE path — a path passed to
   `git -C` is read against the repo dir, not the workspace root, so an absolute path is what
   keeps the worktree under `.worktrees/` (outside the repo's own tree, per rule 3):
   ```
   git -C repos/<repo> worktree add -b <task> "$(pwd)/.worktrees/<repo>/<task>" <base>
   ```
   Done when the command succeeds.

3. **Verify isolation.** `git -C repos/<repo> worktree list` shows the new worktree on branch
   `<task>`. `git status` at the workspace root does NOT list it — `.worktrees/` is gitignored;
   if it appears, the workspace `.gitignore` is broken, so fix it before continuing. Done when
   both hold.

4. **Report.** Give the ready path `.worktrees/<repo>/<task>/` and its branch. Work runs there
   via `git -C .worktrees/<repo>/<task>`, from the workspace root. Done when the caller has the
   path and branch.

## Cleanup

The caller's job once the work lands, not this skill's: `git -C repos/<repo> worktree remove
.worktrees/<repo>/<task>`. Run `git worktree prune` BEFORE deleting a source repo, and
`git worktree repair` if the workspace moves (CLAUDE.md rule 4).
