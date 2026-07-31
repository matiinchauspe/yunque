---
name: aw-review-work
description: Use when the user wants to adversarially stress-test a decision, plan, spec, design, skill, or code change before acting — e.g. "review this work", "revisá esto", "revisá este trabajo". Runs two blind independent reviewers and synthesizes a verdict.
---

# aw-review-work

Two independent reviewers review the same target at once, blind to each other. You
coordinate and synthesize — you never review yourself. The value is independence: two
minds that never saw each other's reasoning catch what one alone misses.

## Invoking this skill

- **The user asked explicitly** ("review this work", "revisá esto") → launch now.
  They already consented.
- **You surfaced it** (a strong decision just landed — see the gate) → emit ONE line
  offering to review, then WAIT. Launching two reviewers is expensive; never launch without
  a yes. If the offer is ignored or declined, do not offer again for that target this
  session.

**Strong-decision gate — offer only when BOTH hold, judged from the conversation:**
1. The target is an architecture/design decision, or a plan/spec/design for a feature
   spanning multiple files/components (not a one-liner).
2. It is about to be acted on (implementation or commit is the stated next step).

**Never offer for** (explicit-only): bug fixes, typos, single-file mechanical edits,
refactors within one unit, doc tweaks, answering questions, exploratory work.

## What to review (the target kind sets the criteria)

Infer the kind from the target. If it fits several, apply the UNION of their criteria; if
none fits, use `decision`.

- **decision / plan** (default) — unstated assumptions, contradictions, uncovered cases,
  "executable without asking a single question?", blast radius, reversibility.
- **spec** — ambiguity, placeholders, testability, completeness.
- **design** — boundaries, does the mechanism degrade, coherence with existing patterns.
- **skill** — trigger fires reliably, no tool names leak, a degrade ladder where it
  declares a tool-backed capability, convention fidelity (`aw-<verb>-<noun>`, information
  hierarchy).
- **code** — correctness, edge cases, error handling, performance, security, conventions.

## The review

1. **Launch two independent reviewers** over the same target, each blind to the other —
   neither knows another exists. Give both the identical target and criteria. They only
   READ and report; they never edit anything (so no branch or workspace isolation is
   needed).
   - If you cannot run them at once, run two blind passes one after the other, each in a
     fresh context so the second never sees the first's reasoning. If a clean fresh context
     cannot be guaranteed, proceed but declare the blindness is best-effort. If even that is
     impossible, say so — never claim a blindness you did not get.
2. **Each reviewer returns findings only** — no praise, no approval. Per finding: severity
   (CRITICAL / WARNING real / WARNING theoretical / SUGGESTION), location, what is wrong
   and why, and the fix intent.
3. **Synthesize** — you, not a reviewer:
   - Confirmed — found by BOTH → high confidence.
   - Suspect — found by ONE → triage.
   - Contradiction — they disagree → flag for a human call.

## Classifying warnings

Per warning, ask whether it is real or theoretical:
- **code** → "can a normal user, using it as intended, trigger this?"
- **non-code** → "would an implementer building from this hit a wrong or blocked outcome?"

Yes → **real**, fix it. No → **theoretical** (labelled INFO in the verdict — same thing);
do not fix, do not re-review.

## Converging

- **Round 1:** present the verdict and ask before fixing. Fix confirmed CRITICALs and real
  WARNINGs, then re-review.
- **Round 2+:** re-review only if confirmed CRITICALs remain. If only real WARNINGs are
  left, fix them inline and trust the fix on your own inspection — no new round. After 2
  fix iterations with issues still left, ask the user before continuing.
- **Suspect and Contradiction findings also block a clean verdict:** a Suspect CRITICAL or
  real WARNING (raised by one reviewer), or an unresolved Contradiction, must be resolved —
  fixed, or dismissed with a stated reason — before APPROVED. Never pass one over just
  because a single reviewer raised it.

**Terminal verdicts — reach exactly one:**
- **APPROVED** = 0 confirmed CRITICALs + 0 confirmed real WARNINGs, and no open Suspect
  CRITICAL or Contradiction. Theoretical warnings and suggestions may remain.
- **ESCALATED** = issues remain after 2 fix iterations; hand them to the user with the open
  findings.

v1 assumes a user is present at these convergence points. An unattended path — driving to a
verdict without prompting — is future work (see the design's AFK note); do not improvise it
here.

## Blocking rules (do not skip)

- Do NOT declare APPROVED until the convergence criteria above are met.
- Do NOT commit, push, or move on after applying fixes until re-review completes.
- Do NOT say "done" until the verdict is terminal.
- YOU never review the target yourself — you only launch, gather, and synthesize.

## Coexistence

Where the global `judgment-day` skill is installed (Claude Code today), it owns the judge
phrasing — AGENTS.md carries the one authoritative list and routes it to the global. This
skill fires via its `auto`/offer path and its own review phrases ("review this work",
"revisá esto", "revisá este trabajo"), which keep it clear of that list. Where the global is
not present, these phrases are the only path. When the global is retired, its phrases move
here.

## Attribution

Distilled from the global `judgment-day` skill — the same blind-parallel protocol,
rewritten tool-agnostic for this harness. Named for what it does: review any work, not only
code (Matt Pocock's `code-review` is code-only), so the verb is `review` and the noun the
general `work`.
