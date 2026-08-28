---
name: yun-research
description: Investigate a question against primary sources and leave a cited Markdown file in the repo. Use when the user wants a topic researched, docs or API facts gathered, a claim verified, or reading legwork delegated while they keep working.
---

Research is **legwork you delegate**, not thinking you outsource: a background agent
does the reading, you keep working, and you get back a cited document to react to.

## Steps

1. **Recall first.** Follow `skills/_shared/memory-convention.md` to recall prior
   research on this question, scoped to the target project. If an existing digest
   already answers it, surface that and stop — no agent needed.

2. **Spin up a background agent** to do the reading, so the user is not blocked.
   Hand it the question and this brief:
   - Work from **primary sources** — official docs, source code, specs, first-party
     APIs — never a secondary write-up of them. Follow every claim back to the
     source that owns it.
   - Write the findings to a **single Markdown file** in the repo, each claim cited
     to its source. Save it where the repo already keeps such notes; match the
     existing convention, and if there is none, pick somewhere sensible and say where.

   Done when the agent has returned and the file exists with a source on every claim.

3. **Persist the digest.** Once the file lands, follow
   `skills/_shared/memory-convention.md` to persist a short digest — the question and
   its findings in a few lines — scoped to the target project. The full file stays in
   the repo; the digest is what downstream skills recall without re-reading it.

4. **Hand it back.** Surface the file to the user. It is something to react to — the
   input that feeds grilling, planning, or a handoff — not a final answer.

## Attribution

Adapted for this workspace from **Matt Pocock's `research`** (github.com/mattpocock/skills,
MIT). Changes: renamed to the `yun-<verb>-<noun>` harness convention; added a
recall-first step and a project-scoped digest via `skills/_shared/memory-convention.md`
(the full cited file still lives in the repo); dropped the OpenAI agent wiring.
