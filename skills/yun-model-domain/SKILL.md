---
name: yun-model-domain
description: Build and sharpen a project's domain model — its ubiquitous language and the decisions behind its shape. Use when the user wants to pin down or resolve a fuzzy, overloaded, or conflicting domain term, agree on canonical vocabulary, or record a hard-to-reverse decision about the domain's shape (an ADR); or when another skill needs to maintain the domain model. This skill writes the glossary and ADRs — reading them for vocabulary is a one-liner any skill does through domain-convention.
---

# yun-model-domain

Domain modeling is the **active** discipline: challenging terms, probing edge cases, and
writing the ubiquitous language and the decisions down the moment they crystallise. Merely
reading the model for vocabulary is not this skill — that is a one-line habit any skill does
through `skills/_shared/domain-convention.md`. **This skill is for when you are changing the
model, not consuming it.**

The model has two kinds, and only two — a **glossary** of terms and a log of **decisions**
(ADRs). Both are captured through `domain-convention.md` (`capture`), which owns where they
land and when they commit; this skill owns what they say. Create files **lazily** — only when a term or a decision
actually resolves. Never scaffold an empty model upfront, and never nag that one is missing.

## During the session

Not a pipeline — a set of reflexes applied continuously while designing. Ask one question at
a time; resolution happens through the exchange.

- **Challenge against the glossary.** When a term collides with the language already
  captured, call it out at once — *"the glossary defines cancellation as X, but you seem to
  mean Y — which is it?"*
- **Sharpen fuzzy language.** On a vague or overloaded word, propose a precise canonical
  term — *"you're saying account — do you mean the Customer or the User?"*
- **Probe with scenarios.** Invent edge-case scenarios that force precision about the
  boundaries between concepts.
- **Cross-check the code.** See whether the code agrees; surface a contradiction — *"the code
  cancels whole Orders, but you just said partial cancellation is possible — which is right?"*
- **Capture on resolution.** The moment a term settles, write it to the glossary; the moment
  a decision qualifies, write the ADR. Capture them as they happen — never batch them up.

## The glossary

Each term is a **bold name**, a tight one-to-two sentence definition of what it *is* (not
what it does), and an `_Avoid_:` line naming the synonyms to reject:

```md
**Invoice**:
A request for payment sent to a customer after delivery.
_Avoid_: Bill, payment request
```

- **Be opinionated.** When several words name one concept, pick the best and list the rest
  under `_Avoid_`.
- **Only project-specific terms.** Before adding one, ask: is this unique to this project's
  domain, or a general programming concept? Only the former belongs.
- **Group under subheadings** when natural clusters emerge; a flat list is fine when they don't.
- A glossary and **nothing else** — no implementation detail, no spec, no scratch pad.

## The decisions (ADRs)

Record an ADR only when all three hold — the choice is **hard to reverse**, **surprising
without context** (a future reader will ask "why this way?"), and **the result of a real
trade-off** (there were genuine alternatives and you picked one for reasons). Most choices
meet none of these; leave them unrecorded.

The format is a short title and one-to-three sentences — context, choice, and why:

```md
# Event-sourced orders

Orders are stored as an append-only event log rather than mutable rows, so any past state can
be reconstructed for audit. Trades write-path simplicity for a full history we need.
```

That is the whole ADR — the value is recording *that* a decision was made and *why*, not
filling out sections. Add a `Status` (`proposed | accepted | deprecated | superseded by
ADR-NNNN`), `Considered Options`, or `Consequences` section only when it earns its place;
most won't.

## Attribution

Adapted for this workspace from **Matt Pocock's `domain-modeling`**
(github.com/mattpocock/skills, Apache-2.0). Changes: the producer/consumer split is
externalised — this skill is the producer, while the rules for *reading* the model (which
Matt keeps in a per-repo `docs/agents/domain.md` plus an inline `tdd` rule) live once in
`skills/_shared/domain-convention.md`; the glossary and ADR formats are inlined rather than
shipped as separate `CONTEXT-FORMAT.md` / `ADR-FORMAT.md` reference files (self-contained,
matching `yun-tdd`); multi-context (`CONTEXT-MAP.md`) is deferred to a documented extension;
renamed to the `yun-<verb>-<noun>` family. The active-modeling reflexes, the glossary format,
and the three-part ADR gate are preserved.
