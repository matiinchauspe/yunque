---
name: yun-grill-plan
description: Use when the user wants to stress-test a plan, decision, design, or idea before acting — or uses any "grill" trigger ("grill me", "grillame", "cuestioname", "desafiame", "poné a prueba esto"). Front-loads the open decisions into one relentless interview so the settled decisions are airtight enough to execute autonomously.
---

# yun-grill-plan

Interview the user **relentlessly** about every aspect of this until you reach a shared understanding. Walk down each branch of the decision tree, resolving dependencies between decisions one by one. For each question, provide your **recommended answer**.

Ask the questions **one at a time**, waiting for feedback on each before continuing. Asking multiple questions at once is bewildering.

## Facts vs decisions

If a *fact* can be found by exploring the environment, **look it up rather than asking** — the filesystem, tools, the codebase, and prior decisions from past sessions are all fair game. Before putting any decision to the user, **recall** prior work on the decision's keywords (follow `skills/_shared/memory-convention.md`): if it was already settled in a prior session, surface it ("you decided X before — still holds?") instead of asking fresh. Nothing relevant found → ask. Any recommended answer you attach is grounded in what the lookup turned up, not a lazy default.

Surface only decisions an implementer couldn't safely default on their own; skip trivia they'd just pick. That bound is what keeps the interview relentless without turning endless.

The *decisions* themselves are the user's. Put each one to them and wait for the answer.

## The language

A plan that lives in a project's domain is grilled in that domain's **ubiquitous language**. Before the first question, invoke `yun-model-domain` and **consult** the model (follow `skills/_shared/domain-convention.md`). Then open with the **terms at stake**: every domain term the plan uses, each beside the glossary's definition — or marked *missing* from it, or *colliding* with it. That list is where the grilling starts: a missing or colliding term is a question like any other, put before the decisions that lean on it.

From there, keep `yun-model-domain`'s session reflexes running alongside every question — challenge, sharpen, probe with a scenario, cross-check the code — and the moment a term resolves, hand it to `yun-model-domain` to
**capture**: it owns the glossary write, as it owns the ADR's. A plan about tooling or the harness itself has no domain to sharpen — grill it on decisions alone.

## As decisions resolve

**Persist** each settled decision as it lands — not at the end — following `skills/_shared/memory-convention.md`. The grilling front-loads the questions so the work afterwards runs autonomously; the record is what makes a later session (or a fresh agent) able to pick it up without re-asking.

A decision that changed the system's **architecture** — how it is built, not just what it does — has earned an ADR: hand it to `yun-model-domain`, which owns whether it qualifies and writes it. In doubt, hand it over. Entered carrying a map's decision ticket, you were dispatched by a walk: the walk records the resolution and owns that hand-off, so grill on.

## Done

Do not act on the plan until the user confirms you have reached a shared understanding. The bar: an implementer could execute the result without asking a single question — and, in a domain grill, could name every concept in the glossary's own words. Close by listing the terms captured this session, or stating that none resolved.

## Attribution

Adapted for this workspace from **Matt Pocock's `grilling`** (github.com/mattpocock/skills, MIT). Change: facts-lookup extended to recall prior decisions, and settled decisions are persisted as they resolve — both via `skills/_shared/memory-convention.md`. The domain-language pass composes `yun-model-domain` into the grill the way Matt's `grill-with-docs` composes `grilling` with `domain-modeling`, but conditionally and model-invoked rather than as a separate hand-typed skill.
