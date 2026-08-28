---
name: yun-build-prototype
description: Build a throwaway prototype to answer a design question by making it concrete and runnable instead of arguing it on paper — a hand-driven state model, or rival UI variants side by side. Use when a decision hangs on "does this logic/state model feel right" or "what should this look like", or the user wants to sanity-check a shape by running it before committing; reach for it over conversation when arguing on paper isn't settling the question. Also when another skill needs a design question resolved with a concrete artifact.
---

# yun-build-prototype

A prototype is **throwaway** code that answers a question — and the question decides its
shape. It raises the fidelity of a decision by making something concrete to react to, instead
of arguing the shape in the abstract. The prototype is disposable from the first line; only
the answer it settles is kept.

## Route by the question

Pick the branch by **which question** is being answered — getting this wrong wastes the whole
prototype. Read the question off the user's prompt, the surrounding code, or by asking if the
user is around.

- **"Does this logic / state model feel right?"** — build it as a hand-driven state model.
  Read `LOGIC.md` for how.
- **"What should this look like?"** — build it as rival UI variants side by side. Read
  `UI.md` for how.

If the question is genuinely ambiguous and the user isn't reachable, default by the
surrounding code — a backend module leans logic, a page or component leans UI — and state the
assumption at the top of the prototype. This step only chooses the path; the build happens in
the branch file, and the skill finishes at capture. Done when the branch is chosen and its
file loaded.

## Shared rules — both branches

- **Throwaway from day one, marked as such.** Locate it beside the module or page it
  prototypes for, but name it so any reader sees at a glance it is a prototype.
- **One command to run.** Whatever the project's task runner already supports; the user
  starts it without thinking.
- **No persistence.** State lives in memory. Persistence is the thing the prototype is
  *checking*, not something it should lean on — add a scratch store only when the question
  itself is about persistence, and name it so it screams "wipe me".
- **Least code that runs.** The point is to learn something fast, so stop at runnable: no
  tests, no abstractions, no error handling beyond what keeps it on its feet.
- **Surface the state.** After every action or variant switch, render the full relevant
  state — the whole value is the human seeing what happened.

## Capture when done

The prototype has done its job the moment the question is answered; now keep the answer and
discard the scaffold.

- **Fold the validated decision into the real code** — only the decision, never the prototype
  itself. Each branch's file says what "the decision" is for its shape.
- **Park the prototype out of the way.** Commit the full prototype on an isolated throwaway
  branch, off the mainline, so the losing variants and the throwaway shell can't rot into the
  real code. The branch is the archive — it outlives any worktree it was built in.
- **Persist the answer.** Persist a short digest — the verdict and the question it settled —
  through `skills/_shared/memory-convention.md`, scoped to the project, so a later session
  recalls what this prototype decided without re-deriving it.

Done when the code reflects the answer to the question, the prototype is committed off the
mainline, and its digest is persisted through the memory convention.

## Attribution

Adapted for this workspace from **Matt Pocock's `prototype`** (github.com/mattpocock/skills,
MIT). Changes: switched to model-invocation with harness trigger phrasing; the two
branches stay as **disclosed reference files** (`LOGIC.md` / `UI.md`) because they are
mutually exclusive paths — a run loads only the branch it takes; capture is rebound to the
harness seams that exist today — the prototype is parked on a throwaway branch off the
mainline and its verdict `persist`ed through `memory-convention` — rather than the source's
"commit + leave a context pointer on the implementation issue", since the harness has no
ticket-resolve seam yet. The question-routes-the-shape spine, the pure-module /
throwaway-shell split, and the structurally-different-variants rule are preserved.
