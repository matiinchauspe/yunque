---
name: aw-grill-plan
description: Use when the user wants to stress-test a plan, decision, design, or idea before acting — or uses any "grill" trigger ("grill me", "grillame", "cuestioname", "desafiame", "poné a prueba esto"). Front-loads the open decisions into one relentless interview so the resulting spec is airtight enough to execute autonomously.
---

# aw-grill-plan

Interview the user **relentlessly** about every aspect of this until you reach a shared understanding. Walk down each branch of the decision tree, resolving dependencies between decisions one by one. For each question, provide your **recommended answer**.

Ask the questions **one at a time**, waiting for feedback on each before continuing. Asking multiple questions at once is bewildering.

## Facts vs decisions

If a *fact* can be found by exploring the environment, **look it up rather than asking** — the filesystem, tools, the codebase, and **engram** (past decisions from previous sessions) are all fair game. Before putting any decision to the user, search engram once (`mem_search` on the decision's keywords, then `mem_get_observation` for the full hit): if it was already settled in a prior session, surface it ("you decided X before — still holds?") instead of asking fresh. Nothing relevant found → ask. Any recommended answer you attach is grounded in what the lookup turned up, not a lazy default.

Surface only decisions an implementer couldn't safely default on their own; skip trivia they'd just pick. That bound is what keeps the interview relentless without turning endless.

The *decisions* themselves are the user's. Put each one to them and wait for the answer.

## As decisions resolve

Save each settled decision to engram (`mem_save`, type `decision`) as it lands — not at the end. The grilling front-loads the questions so the work afterwards runs autonomously; the record is what makes a later session (or a fresh agent) able to pick it up without re-asking.

## Done

Do not act on the plan until the user confirms you have reached a shared understanding. The bar: an implementer could execute the result without asking a single question.

## Attribution

Adapted for this workspace from **Matt Pocock's `grilling`** (github.com/mattpocock/skills, Apache-2.0). Change: facts-lookup extended to engram, and settled decisions are persisted to engram as they resolve.
