---
name: yun-write-pr
description: Write a pull request title and body that show the change — its ticket, its shape, the evidence it works, and how dangerous it is to merge.
disable-model-invocation: true
argument-hint: "Base ref or PR number (default: the branch against its mainline)"
---

# yun-write-pr

A PR body is read by a reviewer deciding whether to merge. Give them **why** the change exists,
its **shape**, the **evidence** it works, and the **merge danger** — nothing they would have to
dig for.

## Steps

1. **Read the change whole.** The commits between base and head (`<base>...<head>`): a PR number
   brings its own base and head; a base ref pairs with `HEAD`; with no argument, `HEAD` against
   its merge base with the mainline. Uncommitted changes are not in the PR: leave them out of the
   body. Read too the spec or ticket the work closes, where
   the branch or the conversation references one, `fetch`ed through
   `skills/_shared/artifact-convention.md`; one that cannot be fetched is noted for the hand-over,
   never a reason to stop — the commits are enough to write from. Done when you can say in one
   sentence why the change exists, and which changed files the Summary will show and which are
   noise.

2. **Take the repo's form.** Look for a PR template the forge reads (`.github/pull_request_template.md`
   or the forge's equivalent) and a title convention or language the repo **declares** — a
   PR-title check in its CI, or its agent instructions or conventions doc asking for one. A
   declared template wins: the sections below fill it — they never replace it. A declared title
   convention or language wins over **The title** below. Domain things go by their canonical
   term: `consult` the domain model through `skills/_shared/domain-convention.md`.

3. **Write the title and the body.** Both forms and their rules are below. No preamble.

4. **Hand it over.** If the user asked to publish it, do so through the forge's own tooling: for
   a branch with no PR, push it and open one with this title and body; for an existing PR,
   replace its title and body. Otherwise hand both back as text.

## The title

`<ticket ID> <description>` — or the description alone when there is no ticket. One form: the ID
is a prefix that appears only when it exists.

```
RES-87 Count the plan limit against the reservation's month
Rename the default domain glossary to GLOSSARY.md
```

**Ticket ID** — only a tracker ID a reviewer can open (`RES-87`, `#42`), written in the tracker's
canonical form: the ticket step 1 fetched, else the one the branch name carries
(`minchauspe/res-87-…` → `RES-87`), else one named in the conversation. A ticket on
`artifact-convention`'s file floor is numbered locally and resolves for nobody else, so it
leaves the title as one with no ticket. The ID is always one that exists — no placeholder, and no
ticket opened just to have one.

**Description** — the outcome of the change, imperative, sentence case, no trailing period, about
72 characters at most. In English, unless the repo declares another language. It starts at the
outcome itself: a conventional-commit prefix (`feat:`, `fix(scope):`) in the repo's history is
habit, not a declaration, so it stays in the commits.

## The template

```markdown
[<ticket or spec>](<url>) | …

<one sentence: why this change exists>

## Summary

<the smallest view of the change>

## Evidence

- **Before:** <what was observed>
  **After:** <what is observed now>

## Merge Danger

**Door:** <one-way | two-way> — <why>

**Blast radius:** <one word> — <what it can touch>
```

**Header** — links a reviewer can open: a tracker URL, a published spec. Never a local path — a
`.yun/` file resolves for nobody else, and a public PR leaks what it names. No such link, no
line. Then the **why** — exactly one sentence: the problem this solves, or what it makes
possible.

**Summary** — the smallest view from [`../yun-show-me/VISUALS.md`](../yun-show-me/VISUALS.md)
that makes the change clear, aimed at the diff: a diff of the shape usually says it fastest, and
a changed schema, endpoint or key type goes first as a contract. One short sentence beside each
view; one view, sometimes two.

**Evidence** — proof the change works, as a before and an after, ranked:

1. A **screenshot** — when the change is visual and the environment can take one.
2. An **execution** — the test that failed and now passes (named, or as pseudocode), the
   command output that changed.
3. **None** — then the section says so in those words, and names what was left unverified.

Report only runs you can point at: one made in this session, one recorded in the ticket's log,
or the ticket's done-signal — else the repo's tests — re-run now for the after. An after with no before is a claim, not evidence.

**Merge Danger** — the **door**: a two-way door walks back cheaply (a revert restores the world);
a one-way door does not — a destructive migration, a published API, a rewritten history, data
sent somewhere. The **blast radius**: everything the merge can reach — consumers that break,
layout that shifts, a flow that degrades on mobile — in one word, then the ramifications.

## Close by stating the verdict

State the PR's link and its title, or that both were handed back as text, and the door in one line —
`two-way, blast radius: local`. A reviewer reads that line first; the body exists to justify it.
Then name what the body could not cover: uncommitted changes left out, a ticket that could not
be fetched.

## Attribution

Adapted for this workspace from **Matt Pocock's `pr`** (github.com/mattpocock/skills, MIT), whose
Summary menu comes from **Dex Horthy's `show-me`** (github.com/humanlayer/skills, MIT); the header
of links, the one-sentence why and the contract view come from **Dex Horthy's `visual-pr`** (same
repo). Changes: the visual menu is read from `yun-show-me`'s `VISUALS.md` instead of copied in;
the repo's own PR template is honoured; the title has its own rule — a tracker ID first when one
exists, no conventional prefix, English unless the repo declares otherwise — where Matt's says
nothing of titles; the glossary is reached through
`skills/_shared/domain-convention.md`; Evidence gained a third rung that says plainly when nothing
was run; and the skill can open the PR, not only draft its body.
