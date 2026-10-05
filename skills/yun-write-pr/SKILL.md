---
name: yun-write-pr
description: Write a pull request body that shows the change — its shape, the evidence it works, and how dangerous it is to merge.
disable-model-invocation: true
argument-hint: "Base ref or PR number (default: the branch against its mainline)"
---

# yun-write-pr

A PR body is read by a reviewer deciding whether to merge. Give them the **shape** of the change,
the **evidence** it works, and the **merge danger** — nothing they would have to dig for.

## Steps

1. **Read the change whole.** The diff and commits against the base — the argument names it,
   else the branch's merge base with the mainline — plus the spec or ticket it closes, `fetch`ed
   through `skills/_shared/artifact-convention.md` when one exists. Done when every changed file
   is either in the Summary's view or deliberately left out as noise.

2. **Take the repo's form.** `conform` through `skills/_shared/practice-convention.md`: a PR
   template or title convention the repo declares wins, and the three sections below fill it —
   they never replace it. Domain things go by their canonical term: `consult` the domain model
   through `skills/_shared/domain-convention.md`.

3. **Write the body.** The template and its rules are below. No preamble.

4. **Hand it over.** If the user asked to publish it, do so through the forge's own tooling: for
   a branch with no PR, push it and open one with this body; for an existing PR, replace its
   body. Otherwise hand the body back as text.

## The template

```markdown
## Summary

<the smallest view of the change>

## Evidence

- **Before:** <what was observed>
  **After:** <what is observed now>

## Merge Danger

**Door:** <one-way | two-way> — <why>

**Blast radius:** <one word> — <what it can touch>
```

**Summary** — the smallest view from [`../yun-show-me/VISUALS.md`](../yun-show-me/VISUALS.md)
that makes the change clear, aimed at the diff: a diff of the shape usually says it fastest. One
short sentence beside each view; one view, sometimes two.

**Evidence** — proof the change works, as a before and an after, ranked:

1. A **screenshot** — when the change is visual and the environment can take one.
2. An **execution** — the test that failed and now passes (named, or as pseudocode), the
   command output that changed.
3. **None** — then the section says so in those words, and names what was left unverified.

Report only what was run in this work. An after with no before is a claim, not evidence.

**Merge Danger** — the **door**: a two-way door walks back cheaply (a revert restores the world);
a one-way door does not — a destructive migration, a published API, a rewritten history, data
sent somewhere. The **blast radius**: everything the merge can reach — consumers that break,
layout that shifts, a flow that degrades on mobile — in one word, then the ramifications.

## Close by stating the verdict

State the PR's link, or that the body was handed back as text, and the door in one line —
`two-way, blast radius: local`. A reviewer reads that line first; the body exists to justify it.

## Attribution

Adapted for this workspace from **Matt Pocock's `pr`** (github.com/mattpocock/skills, MIT), whose
Summary menu comes from **Dex Horthy's `show-me`** (github.com/humanlayer/skills, MIT). Changes:
the visual menu is read from `yun-show-me`'s `VISUALS.md` instead of copied in; the repo's own PR
form is honoured through `skills/_shared/practice-convention.md`; the glossary is reached through
`skills/_shared/domain-convention.md`; Evidence gained a third rung that says plainly when nothing
was run; and the skill can open the PR, not only draft its body.
