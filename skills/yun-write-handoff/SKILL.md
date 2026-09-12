---
name: yun-write-handoff
description: Compact the current session into a disposable handoff baton for the next agent, pointing at durable memory instead of duplicating it.
disable-model-invocation: true
argument-hint: "What will the next session focus on?"
---

# yun-write-handoff

Write a handoff document that lets a fresh agent continue this work without re-deriving it. Save it as one markdown file at `.yun/handoffs/handoff-<slug>.md`. The directory is conventional, so the slug alone points the next session at the baton. **The slug names the effort, not the session** — one effort, one file, however many sessions it takes. It is a disposable **baton**, not a versioned artifact.

## What goes in the baton

- **The world-state it describes, first.** Open with the state that makes the rest true — the commit it was written against, where the work has one — and tell the reader to verify it before trusting anything below. The baton is read by an agent that does not load this skill, so the instruction has to travel inside the baton.
- **Labelled pointers, not copies.** Settled facts already captured — memory references (ids, topic-keys), specs, commits, diffs — get a one-line label plus a reference (e.g. `#502 — chosen rate-limit algorithm`), never a restatement. The label lets the next agent triage without opening every link.
- **The live delta, in full.** What exists only in this conversation: half-done edits, and the open threads you are handing forward (even if you also persisted them). This is the next agent's marching order — write it out, do not reduce it to a pointer.
- **Suggested skills.** Which harness skills fit the open work — e.g. `yun-grill-plan` for an unresolved decision, `yun-write-skill` when authoring.

## Persist the durable part first

Before writing the baton, **persist** anything durable following `skills/_shared/memory-convention.md` — including a full end-of-session summary when the session warrants one. That persistent memory is the long-term store; the baton is the short-lived transfer. Once the durable facts are persisted, the baton shrinks to a pointer plus the live delta — which is the whole point.

## Retire the superseded baton

`.yun/handoffs/` holds live work only. **List it before you name anything and default to a path already there** — writing over that path is what retires the old baton; a fresh name silently keeps both. Invent a slug only when nothing in the list covers this work.

**Name the baton after the effort, not the session.** The next ticket, question, or phase of the same map is the same effort — it takes that baton's path. A second file for one effort is the defect.

A stale baton beside a fresh one is the expensive failure: the next session reads the loser as fact. Retiring one whose effort has closed for good is the user's call — name the file and ask.

## Close by stating what you wrote

State two things the user can refute at a glance:

- **the slug you wrote, and whether it replaced an existing baton** — if you invented a new one, what in the listing you ruled out
- **the ids the baton points at** — where nothing was persisted there is nothing to point at, and the baton carries it in full.

Both failures this catches are silent: a create looks like an overwrite, and a copy looks like a pointer.

## Guardrails

Keep secrets out — the baton becomes the next agent's prompt, so replace every API key, password, or personal detail with a short note of what it was.

If the user passed an argument, treat it as the next session's focus and shape the baton toward it.

## Attribution

Adapted for this workspace from Matt Pocock's `handoff` (github.com/mattpocock/skills, MIT). Changes: durable content is persisted to and referenced from the workspace memory convention rather than restated, so the baton stays a thin pointer-plus-delta; the baton opens with the world-state it describes; and the write closes by stating the slug it wrote and the ids it points at.
