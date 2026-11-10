---
name: yun-clean-worktree
description: Sweep the workspace's worktrees — remove the ones whose work is merged, and ask about each one that is not.
disable-model-invocation: true
---

# yun-clean-worktree

The teardown half of `yun-spawn-worktree`. Work lands by a merge on the forge, with no agent
present, so nothing removes a worktree when its work is done and `.worktrees/` only grows. This
sweep is that removal, run by hand. What is **merged** goes without asking; what is not stays
unless the human says otherwise. It removes **workspaces, never refs**: every branch stands, so
anything committed can be spawned again.

Run at the workspace root and operate with `git -C <repo>` (AGENTS.md rule 1), where a **repo** is
each one in `repos/` and the workspace's own. The sweep covers every worktree a repo registers under
`.worktrees/`; a source checkout is never one.

## Merged

A worktree is **merged** when either proof holds for its branch. Anything else is unmerged.

- **A — the forge.** The branch's PR reads MERGED:
  `gh pr list --head <branch> --state merged` run from the repo. This proof covers squash and
  rebase merges, whose commits never reach the mainline.
- **B — git.** The branch has at least one commit of its own, and every one of them is in the
  repo's **mainline**, resolved by the rule in `skills/_shared/execution-convention.md`, without
  fetching. Its own commits start where the branch was created: the oldest entry of
  `git reflog show <branch>` ("Created from …"). A tip still at that commit has none — that is a
  worktree whose work is all uncommitted, and it is unmerged. Without that "Created from" entry,
  B cannot be shown, and the worktree is unmerged.

A stale mainline ref or an unreachable forge can only turn a merged worktree into an
unmerged one, which gets asked about. No proof failure can delete unmerged work.

## Steps

1. **Inventory.** For each repo, read `git worktree list --porcelain` and keep the
   entries under `.worktrees/`. For each, record: path, branch, the PR state (MERGED, OPEN, CLOSED
   or none), whether proof A or B holds, the uncommitted files (`git status --porcelain`), its size,
   and its **age**. Age is the time since the later of two dates: the tip commit, and the newest
   mtime among its uncommitted files. For each unmerged row, also record:
   - **Its ticket.** A branch a walk cut (`<prefix>/<feature>/<NN>-<nonce>`) names its feature and
     ticket number: `fetch` that implementation ticket through
     `skills/_shared/artifact-convention.md`. A branch carrying the tracker's own id instead
     (`<user>/<id>-<slug>`) is looked up by that id through the same tracker capability. Record
     the ticket's state and, when it is a child, its parent's. A ticket the contract reports
     unreachable, or a branch naming none, is recorded as "no ticket": that `fetch` stop ends
     this one lookup, never the sweep.
   - **Whether it is in use.** A file in it changed within the last hour, or a running process
     has a path inside it. A walk in flight shows up here, so name the walk when the branch is one
     of its links.
   - **Whether its commits exist only here.** No remote branch contains its tip
     (`git branch -r --contains <tip>` is empty).

   A directory under `.worktrees/` holding a `.git` file that no repo registers is an orphan;
   record it and leave it alone. The directories that only nest a worktree (`.worktrees/<repo>/aw/`)
   are neither. Done when every registered worktree has a row.

2. **Remove the merged ones.** In one command: `git -C <repo> worktree remove --force <path>` for
   each merged row, with `<path>` absolute — `-C` reads a relative one against the repo — then
   `git -C <repo> worktree prune` for each repo touched. Uncommitted and ignored files go with it,
   because its work is already merged. Then delete the directories under `.worktrees/` this left
   empty — a directory holding only `.DS_Store` counts as empty. A removal the environment holds
   for the human is theirs to release: hand back that one command, or the id its refusal names,
   and go on to step 3. Done when no merged row is still registered, or the human holds the one
   action that removes them all.

3. **Ask about the unmerged ones, oldest first, one at a time.** Show the path, branch, age, PR
   state, size, its ticket's state, and the uncommitted files, listed when there are any. Warn
   when it is in use, and when its commits exist only here — the branch survives the removal, but
   a lost disk takes it. Recommend delete or keep, with the reason: a worktree in use, or whose
   uncommitted files are the only copy of its work, is a keep. A walk's earlier link is a keep
   too, even when its commits are inside the newer one, because the walk removes that link itself
   when it chains, and a link already gone fails that removal. Then ask:
   delete it or keep it. On delete, remove it as in step 2. Done when every
   unmerged row has the human's answer.

4. **Report.** Four lists: removed as merged (with the proof and any uncommitted files
   discarded), held for the human (merged, or deleted on their answer), removed by the human's
   answer, and kept. Add the orphans, the space freed, and every branch whose commits exist only
   on this machine. Done when every row from step 1 sits on exactly one list.
