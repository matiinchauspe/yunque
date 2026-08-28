---
name: yun-write-handoff
description: Compact the current session into a disposable handoff baton for the next agent, pointing at durable memory instead of duplicating it.
disable-model-invocation: true
argument-hint: "What will the next session focus on?"
---

# yun-write-handoff

Write a handoff document that lets a fresh agent continue this work without re-deriving it. Save it as one markdown file (`handoff-<slug>.md`) in the OS temporary directory and report its path — that path is how the next session gets pointed at the baton. It is a disposable **baton**, not a versioned artifact.

## What goes in the baton

- **Labelled pointers, not copies.** Settled facts already captured — memory references (ids, topic-keys), specs, commits, diffs — get a one-line label plus a reference (e.g. `#502 — chosen rate-limit algorithm`), never a restatement. The label lets the next agent triage without opening every link.
- **The live delta, in full.** What exists only in this conversation: half-done edits, and the open threads you are handing forward (even if you also persisted them). This is the next agent's marching order — write it out, do not reduce it to a pointer.
- **Suggested skills.** Which harness skills fit the open work — e.g. `yun-grill-plan` for an unresolved decision, `yun-write-skill` when authoring.

## Persist the durable part first

Before writing the baton, **persist** anything durable following `skills/_shared/memory-convention.md` — including a full end-of-session summary when the session warrants one. That persistent memory is the long-term store; the baton is the short-lived transfer. Once the durable facts are persisted, the baton shrinks to a pointer plus the live delta — which is the whole point.

## Guardrails

Keep secrets out — the baton becomes the next agent's prompt, so replace every API key, password, or personal detail with a short note of what it was.

If the user passed an argument, treat it as the next session's focus and shape the baton toward it.

## Attribution

Adapted for this workspace from Matt Pocock's `handoff` (github.com/mattpocock/skills, MIT). Change: durable content is persisted to and referenced from the workspace memory convention rather than restated, so the baton stays a thin pointer-plus-delta.
