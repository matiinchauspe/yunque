# LOGIC branch — hand-drive a state model

For the kind of design that looks reasonable on paper but only feels wrong once you push it
through real cases. Build a tiny interactive terminal app that lets the user drive the state
model by hand, one action at a time, until it either feels right or breaks.

## The load-bearing split: portable module, throwaway shell

Put the actual logic behind a small, **pure** interface that could be lifted out and dropped
into the real codebase later. The TUI around it is throwaway; **the logic module is not** —
it is the one thing that survives.

- **Pick the shape that best fits the question** — a pure reducer, a state machine, a set of
  pure functions, or a small class/module — *not* whichever is easiest to wire to a terminal.
- Keep the module free of any I/O: it takes state and an action, returns new state. The shell
  is the only place that reads keys and prints.

## The shell

The shell is throwaway; keep it to one entry point the user drives by keystroke, reading keys
and printing — all behaviour lives in the pure module.

- **Re-render the whole frame each tick** — clear, then reprint the current state and the
  keyboard shortcuts available now. That per-tick redraw is where the shared *surface the
  state* rule lands for this branch.

## The payoff

Drive real cases through it by hand and watch for the human's *"wait, that shouldn't be
possible"* moment. Those are bugs in the **idea** — the whole reason to prototype the logic
before building on it. The prototype is done when the model has been pushed through the cases
that worried you and either holds up or reveals the flaw.
