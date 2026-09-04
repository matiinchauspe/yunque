---
name: yun-diagnose-bug
description: Diagnosis loop for hard bugs and performance regressions — build a tight feedback loop that goes red on the bug before theorising about it. Use when the user says "diagnose this bug" / "diagnosticá" / "no entiendo por qué falla" / "está lento", when a bug resisted the obvious fix, when its repro will not hold still, or when something got slower.
---

# yun-diagnose-bug

A discipline for hard bugs. The whole skill rests on one move: get a **tight** feedback loop
that goes **red** on this bug before you theorise about it. Skip a step only with an explicit
reason, and never step 1.

Before you start, read what the project already knows. **Recall** prior diagnoses of this
symptom through `skills/_shared/memory-convention.md` — a past cause and the loop that caught
it is a head start on step 3's ranking, never a licence to skip building the loop. And
**consult** the project's domain model through `skills/_shared/domain-convention.md` so you
name modules and behaviour in its **ubiquitous language**, and respect the ADRs in the area
you're touching. Both degrade silently when there is nothing to read.

## Redact

This skill has you show commands, outputs and captured artifacts. **Redact every secret
first**: write `<REDACTED>` in its place. Build loops against env vars so the credential stays
in the environment rather than in what you show, and quote only the lines of a captured
artifact that carry the signal. If what survives redaction is not enough to diagnose the bug,
say so and ask the user.

Captured artifacts — traces, HAR files, log dumps, core dumps — carry live secrets whatever you
quote from them. Write them to a **gitignored scratch path** outside the working tree, and
delete them in step 6: the commit that lands the fix must not be able to reach one.

## Steps

### 1. Build a feedback loop

**This is the skill.** With a **tight** pass/fail signal — one that goes **red** on _this_
bug — you will find the cause; bisection, hypothesis-testing and instrumentation all just
consume it. Spend disproportionate effort here, and be **relentless**.

**Pick a route to construct one.** [`LOOPS.md`](LOOPS.md) holds the eleven, ordered — read it
and pick before you build, because the route sets the ceiling on how **tight** the loop can get.
Two of them carry this skill's tails: the **measurement harness**, which records a **baseline**
and goes **red** past a threshold, is the only route a performance regression has, and the
**human-in-the-loop** script is the last resort when no route runs unattended.

**Tighten it.** Treat the loop as a product: faster (cache setup, skip unrelated init, narrow
the scope), sharper (assert the specific symptom, not "didn't crash"), more deterministic (pin
time, seed RNG, isolate the filesystem, freeze the network). A 30-second flaky loop is barely
better than none; a 2-second deterministic one is a superpower.

**When it won't reproduce cleanly.** The goal is not a clean repro but a **higher reproduction
rate**. Loop the trigger 100×, parallelise, add stress, narrow timing windows, inject sleeps. A
50%-flake bug is debuggable; a 1% one is not — keep raising the rate until it is.

**When it won't reproduce at all.** Stop and say so. List what you tried, then ask the user for
one of: access to an environment that reproduces it; a redacted captured artifact — HAR file,
log dump, core dump, timestamped screen recording; or permission to add temporary production
instrumentation. Wait for one of those rather than reasoning your way to a cause you cannot
observe.

**No loop, no step 2.** If you catch yourself reading code to build a theory before that
command exists, stop — jumping straight to a hypothesis is the exact failure this skill exists
to prevent.

Done when you can name **one command** you have already run at least once — showing the
invocation and its redacted output — that is:

- [ ] **Red-capable** — it drives the real bug code path and asserts the user's exact symptom,
      so it goes red on this bug and green once fixed. Not "runs without erroring". For a
      regression, red is a measurement breaching its threshold against the baseline.
- [ ] **Deterministic** — the same verdict every run (flaky bugs: a pinned, high reproduction
      rate).
- [ ] **Fast** — seconds, not minutes.
- [ ] **Agent-runnable** — runnable unattended; a human inside it only via the human-in-the-loop
      script.

Or, when it will not reproduce at all, done when you have stopped and asked — attempts listed,
one of the three asks above made.

### 2. Reproduce and minimise

Run the loop and watch it go **red**. Confirm it produces the failure the **user** described
rather than a different one nearby — wrong bug, wrong fix — that it reproduces across runs, and
that you have captured the exact symptom for later steps to verify a fix against.

Then shrink the repro to the smallest scenario that still goes red: cut inputs, callers,
config, data and steps **one at a time**, re-running the loop after each cut. Keep the
un-minimised loop as you go — step 5 re-runs it. It pays twice: fewer moving parts left to
suspect in step 3, and the clean regression test in step 5.

On a flaky loop, one green run after a cut is luck rather than evidence. Re-run at the
reproduction rate you pinned in step 1 — enough runs that a still-load-bearing element would
have gone red — before you accept the cut.

**Performance branch.** Scale is itself part of the signal: cutting data volume, iteration
count or concurrency takes the measurement harness under its threshold without removing a
cause. Pin the scale that reproduces the regression, minimise around it, and treat that pinned
scale as load-bearing.

Done when the loop has gone red on the **user's** symptom across runs with that symptom
captured, the un-minimised loop is still runnable, and every remaining element is load-bearing:
removing any one of them turns the loop green.

### 3. Hypothesise

Generate **3–5 ranked hypotheses** before testing any of them — generating one at a time
anchors you on the first plausible idea. Each must be falsifiable, stating its prediction: "if
X is the cause, then changing Y makes the bug disappear / changing Z makes it worse". A
hypothesis with no prediction is a vibe: sharpen it or drop it.

Show the ranked list to the user before testing. They re-rank it instantly on domain knowledge
("we just deployed a change to #3") or name the ones they have already ruled out. Don't block
on it — proceed with your own ranking if the user is away.

Done when the ranked, falsifiable list exists and has been shown.

### 4. Instrument

Each probe maps to a specific prediction from step 3, and you **change one variable at a
time**. Reach for a debugger or REPL first where the environment supports one — one breakpoint
beats ten logs — then targeted logs at the boundaries that distinguish the hypotheses.

**Tag every debug log** with a unique prefix, e.g. `[DEBUG-a4f2]`, so cleanup is a single
search. Untagged logs survive; tagged logs die.

**Performance branch.** For regressions, logs are usually the wrong instrument: measure against
step 1's baseline, then bisect. Measure first, fix second.

Done when a probe has confirmed one hypothesis, or falsified all of them and sent you back to
step 3 with what it taught you. Three round trips without a confirmed hypothesis is the signal
to stop, report what you have ruled out, and clean up through step 6.

### 5. Fix, test-first

Write the regression test **before** the fix, but only where a **correct seam** exists: one at
which the test exercises the real bug pattern as it occurs at the call site. A seam too shallow
to replicate the chain that triggered the bug gives false confidence.

**No correct seam is itself the finding.** Report it — the codebase's shape is what keeps this
bug from being locked down. Still fix it, and say plainly that the lockdown is missing and why.

Where a correct seam does exist, **name it as the candidate and carry it into `yun-tdd`** — the
minimised repro shows where the bug's chain actually runs, so the seam arrives with its
evidence rather than being re-derived. A candidate is not an agreement: `yun-tdd`'s rule
stands, and **the user** confirms the seam there before any test is written. Unlike step 3's
ranking, this gate **blocks** — a test at an unconfirmed seam is the false confidence this step
opened by warning about. Then turn the minimised repro into a failing test at the confirmed
seam and drive the fix through `yun-tdd`'s **red → green** loop, which owns the seam discipline
and what makes a test worth keeping. Finally re-run step 1's loop against the original,
un-minimised scenario.

**Performance branch.** A timing assertion at a seam is a flaky test, and `yun-tdd`'s
**red → green** loop has no baseline or threshold to drive. Step 1's measurement harness is
already the regression check: fix, then run it against the recorded baseline until it goes
green under the threshold, and report the before and after numbers.

Done when one of:

- **Seam confirmed** — the regression test is written at it, it passes, and step 1's loop is
  green against the un-minimised scenario.
- **No correct seam** — its absence is reported with the reason the codebase's shape blocks the
  lockdown, and step 1's loop is green against the un-minimised scenario without one.
- **Performance regression** — step 1's measurement harness is green under its threshold against
  the recorded baseline, with the before and after numbers reported.
- **Seam unconfirmed, user away** — nothing is written at it. Park: report the confirmed cause
  and the seam candidate, and leave the fix for the confirmation. The gate holds; the run ends
  rather than stalling behind it.

### 6. Clean up

- [ ] Every `[DEBUG-...]` probe is removed — search the prefix.
- [ ] Throwaway harnesses and any diagnostic test step 1 wrote are deleted — or, when the run
      ends without a fix, moved to scratch and their paths reported, so the run that resumes
      inherits the loop instead of rebuilding it.
- [ ] Every captured artifact is deleted from scratch.
- [ ] The hypothesis that proved correct — or, when step 4 aborted, the ones it ruled out — is
      `persist`ed through `skills/_shared/memory-convention.md` with the symptom and the loop
      that caught it, so the next run's recall starts where this one ended.

The fix lands where an attended run lands anywhere in this harness: commit it only when the
user asks.

Done when every box is checked.

## Attribution

Adapted for this workspace from **Matt Pocock's `diagnosing-bugs`**
(github.com/mattpocock/skills, MIT). Changes: renamed to the `yun-<verb>-<noun>` convention; the
phases become numbered steps, each closing on a **Done when** criterion; reading `CONTEXT.md` and
ADRs becomes a `consult` through `skills/_shared/domain-convention.md`, and the correct hypothesis
a `persist` through `skills/_shared/memory-convention.md`, with a `recall` at the head to read it
back. The write-test-watch-it-fail-apply-fix loop is handed to `yun-tdd`, the single source of
truth for seams and test quality. A measurement route is added to give the performance branch a
path through step 1, and the routes are disclosed to [`LOOPS.md`](LOOPS.md). Named tools, the
`hitl-loop` template and the `agents/openai.yaml` wiring are dropped, since the harness runs
against repos whose stack it does not choose. Upstream's "never log everything and grep" is
restated positively per this harness's negation rule, and Spanish triggers are added to the
description. The loop-first discipline, the **tight** and **red** leading words, minimisation,
ranked falsifiable hypotheses, tagged probes, the performance branch and the cleanup checklist
are preserved.
