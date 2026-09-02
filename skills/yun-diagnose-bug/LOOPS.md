# Routes to a red loop

Eleven ways to construct step 1's feedback loop, in roughly the order to reach for them. The
route sets the ceiling on how **tight** the loop can get, so read past the first one that could
work before committing to it.

1. **Failing test** at whatever seam reaches the bug — unit, integration, e2e. It is a
   diagnostic loop that step 6 retires; step 5 writes the regression test, at a confirmed seam.
2. **Curl / HTTP script** against a running dev server.
3. **CLI invocation** with a fixture input, diffing stdout against a known-good snapshot.
4. **Headless browser script** driving the UI, asserting on DOM, console or network.
5. **Replay a captured trace** — save a real request, payload or event log to scratch and
   replay it through the code path in isolation.
6. **Throwaway harness** — a minimal subset of the system, deps mocked, reaching the bug in
   one function call.
7. **Property / fuzz loop** — for "sometimes wrong output", run 1000 random inputs and look
   for the failure mode.
8. **Bisection harness** — if the bug appeared between two known states, automate "boot at
   state X, check, repeat" so `git bisect run` can consume it.
9. **Differential loop** — the same input through two versions or configs, diffing outputs.
10. **Measurement harness** — for a regression, where there is no binary failure to assert.
    Record a **baseline**, time the path against it, and go **red** past a threshold you set.
    Reach for a profiler or query plan where the timing alone won't localise it.
11. **Human in the loop** — last resort. Generate a bash script that walks the human through
    the steps only they can perform and captures their output back to you, so the loop stays
    structured even with a person inside it.
