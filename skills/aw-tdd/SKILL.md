---
name: aw-tdd
description: Test-driven development — the red → green loop, tests written only at pre-agreed seams. Use when the user wants to build a feature or fix a bug test-first, or mentions TDD or red-green. Reached by the implement flow to drive a build at a seam.
---

# aw-tdd

TDD is the **red → green** loop: a failing test first, then only enough code to pass it.
This skill is the reference that makes the loop produce tests worth keeping — what a good
test is, where it goes, the anti-patterns, and the rules of the loop. Every section applies
on **every** cycle; consult them before and during the loop, not after.

When exploring the codebase, read the project's domain glossary or context doc if one
exists so test names and interface vocabulary match its **ubiquitous language**, and respect
the ADRs in the area you're touching.

## What a good test is

A good test verifies **behavior through a public interface**, never an implementation
detail. Code can change entirely; the test shouldn't. It reads like a specification — "user
can checkout with a valid cart" says exactly what capability exists — and survives refactors
because it doesn't care about internal structure. Prefer real collaborators; **mock only at
an external boundary you don't own** (network, clock, filesystem), never an internal one.

## Seams — where tests go

A **seam** is the public boundary you observe behavior at without reaching inside. Tests
live at seams, never against internals.

**Test only at pre-agreed seams.** Before writing any test, write the seams under test down
and confirm them with the user — no test is written at an unconfirmed seam. When the work
came through a spec, those seams are already named in its Testing Decisions
(`aw-write-spec`); use them. You can't test everything: agreeing the seams up front is how
effort lands on the critical paths and complex logic instead of every edge case. Ask:
"what's the public interface, and which seams should we test?"

## Anti-patterns

- **Implementation-coupled** — mocks internal collaborators, tests private methods, or
  verifies through a side channel (querying the database instead of using the interface).
  The tell: the test breaks under a refactor when behavior hasn't changed.
- **Tautological** — the assertion recomputes the expected value the way the code does
  (`add(a, b)` asserted equal to `a + b`, a hand-derived snapshot, a constant equal to
  itself), so it passes by construction and can never disagree with the code. Expected
  values come from an **independent** source of truth — a known-good literal, a worked
  example, the spec.
- **Horizontal slicing** — all tests first, then all implementation. Bulk tests verify
  _imagined_ behavior: you test the shape of things, the tests go insensitive to real
  changes, and you commit to test structure before understanding the implementation. Work in
  **vertical slices** — one test → one implementation → repeat, each test a **tracer bullet**
  that responds to what the last cycle taught you.

## Rules of the loop

- **Red before green.** Write the failing test first, then only enough code to pass it.
  Don't anticipate future tests or add speculative features.
- **One slice at a time.** One seam, one test, one minimal implementation per cycle.
- **Refactor once green.** With the test passing, clean up the code before the next slice —
  the third beat of each slice, riding on a test that must stay green. The red → green part
  stays a plain failing-test-then-pass rhythm.
- **Run tests tightly.** Typecheck and the single test file under change on every cycle; the
  full suite once, at the end.

## Attribution

Adapted for this workspace from **Matt Pocock's `tdd`** (github.com/mattpocock/skills,
Apache-2.0). Changes: renamed to the `aw-<verb>-<noun>` harness family (the `TDD` leading
word is kept intact for invocation reliability); the TypeScript-specific `tests.md` /
`mocking.md` reference files are dropped, their mocking guidance collapsed to one
language-agnostic rule; refactor stays in the loop as each slice's third beat once green
(the source routes it to a review stage) and seams point to `aw-write-spec`'s Testing
Decisions instead of the source's sibling skills. The red → green loop, the good-test definition, the three
anti-patterns, and the seam discipline are preserved.
