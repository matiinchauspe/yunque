# Evals — does the skill that should fire, fire?

`lenses/` is a linter for the harness **text**. This is the other instrument: it measures
**behaviour**, by running a real agent against the real harness and reading what it reached
for. A skill can be perfectly consistent, perfectly indexed, perfectly named, and never fire.

```
evals/run                          # every skill, 5 runs per case
evals/run --runs 3 yun-review-work # one skill, fewer runs
```

Exit: `0` measured · `1` measured, and some skill fired **0 of N** · `2` **not measured** —
a gate failed and no reading was printed.

**There is deliberately no threshold between 0 and 1.** This instrument reports a number, not
a verdict, and inventing a passing mark would be inventing a finding. Zero is the one
non-arbitrary point on the scale: a skill that will not fire on its own declared phrasing will
not fire on how a person talks.

## It does not measure the harness

It measures a **runner reading** the harness. Behaviour only exists inside a tool, so there is
no tool-agnostic way to observe it. Three pieces, and only one of them is tool-specific:

| Piece | Tool-agnostic? | |
| --- | --- | --- |
| `cases.tsv` | **yes** | pure harness vocabulary — a prompt and the skill it should reach |
| the runner | no, and **declared** | the only proprietary part, isolated so it can be replaced |
| the verdict | **names its runner and model** | it never speaks for "the harness" in general |

A second runner is a second reading, and the two can **disagree** — that disagreement is the
information. This workspace's tool-agnostic claim lives in prose and has never been measured;
`cursor-agent` offers the same `-p` / `stream-json` shape and is where that measurement would
come from. It is deliberately unbuilt and unverified: building on a world nobody has read is
the most expensive mistake this workspace has recorded.

## What it reports

A percentage per skill. It is a **regression** instrument, not a grade — *"is 70% good?"* is
unanswerable, while *"was 70%, changed the description, now 90%"* is actionable. **The delta
is the whole product.**

Which is why `cases.tsv` is **frozen** and its hash is printed beside every reading. Two
readings taken against different corpora are not comparable, and the hash is how you can tell.
Regenerating the corpus **resets the baseline**: do it deliberately, never as a side effect of
editing a skill. If the cases were derived from the descriptions at run time, improving a
description would move the skill and its test together and the delta would mean nothing.

## Fired means: in the first batch

The expected skill must appear in the agent's **first batch of tool calls** — what it reached
for before seeing a single result.

- **Why not "anywhere in the run":** `yun-write-skill` states the root virtue as *"the agent
  taking the same **process** every run"*. A skill invoked after the work is done governed
  nothing; a process that already happened cannot be governed retroactively. Counting a late
  firing as a pass would read green over exactly that.
- **Why the batch and not the first block:** an assistant message can carry several tool calls
  at once, chosen simultaneously, before any result came back. That is one decision, and a
  skill sitting inside it was reached for first.
- **`Bash` counts as an action even when the command is `git status`.** Classifying shell
  commands read-vs-write is model judgment, and an instrument that needs a model to read its
  own output is the guard needing the same treatment as the thing it guards.

## The corpus measures declared phrasing, not natural speech

Every case comes from what the skill **already declares** in its own `description` —
`quoted` where the description quotes a phrase, `derived` where it names the branch in prose
and the wording is a faithful reading of it. The column is in `cases.tsv` because the two carry
different weight: **a failure on a quoted phrase is unambiguous** — the skill will not fire on
its own literal words.

This is the skill's own answer key, and grading against it is the cheap half of the question.
It survives as v1 for three reasons: it costs nothing to write, it is the **upper bound** (a
skill that will not fire on its declared phrase will not fire on natural speech), and **it
already discriminates** — the first measurement found a skill at 1 in 6 on a phrase quoted
verbatim in its own description.

The harder corpus — how a person actually asks — is worth writing **only once this one stops
discriminating**, and not before.

Only the **12 model-invocable** skills appear. The four carrying `disable-model-invocation`
cannot be fired by a model at all.

## Calibration — four gates, and nothing is reported if one fails

A reading nobody calibrated is not a reading. Gate 0 costs no API calls at all.

| Gate | Asks | Fails when |
| ---- | ---- | ---------- |
| **0 · the reader** | does the transcript reader read planted transcripts correctly? | any fixture disagrees, **or there are none** |
| **1 · can it see a firing?** | a prompt naming a skill outright must be read as a firing | the reader or the runner changed |
| **2 · did the corpus load?** | the runner's `init` event must list all 12 skills | the corpus measured is not the corpus on disk |
| **3 · does it invent firings?** | a prompt with no business firing anything must fire nothing — **and the scorer, asked a question whose answer must be 0, must answer 0** | the instrument reports firings that did not happen |

**Gate 1 asks nothing about harness quality** — the prompt names the skill outright. Keeping
instrument health separate from the reading is what stops a real harness regression from being
mis-reported as a broken instrument.

**Gate 3 runs the scorer**, not a private copy of its logic. A scorer exercised only by the
measurement is one that nothing can catch going wrong: mutate it to always answer "fired" and
the report reads as a flawless harness. That is the silent direction, the one a green report
hides, and gate 3 is the only thing that sees it.

**Gate 0 collapses absent and empty into one verdict** — a fixtures directory that exists and
holds nothing would otherwise run the loop zero times and pronounce the reader trustworthy.
That exact hole was found inside this workspace's other calibration gate.

## The fixtures, and what each one actually proves

Fixtures test the **deterministic half** — reading a transcript. Unlike the measurement, that
half can be proven without spending a single API call, which is why gate 0 runs first.

| Mutation of the reader | Caught by |
| ---------------------- | --------- |
| reads the first **block** instead of the first batch | **`parallel-first-batch` alone** |
| drops the complete-line guard | **`half-flushed-line` alone** |
| never stops, so it reads the **last** batch | **`bash-first` alone** |
| tool name read off by one | `bash-first` · `parallel-first-batch` · `skill-first` |
| skill argument read off by one | `parallel-first-batch` · `skill-first` |
| loop starts at 1 and invents an entry | `bash-first` · `parallel-first-batch` · `skill-first` |

**Stated plainly rather than claimed away: two of the five are not sole catchers.**
`skill-first` is the mandatory clean case — the one that catches a reader gone greedy, and the
suite's own rule for adding a check requires one. **`no-tool-call` catches nothing in this
set.** It is kept because an agent answering in prose is a real outcome and `probe` depends on
that reading as empty, but it has never been observed to fail; if it never earns a catch it
should be dropped rather than left as decoration.

**Three traps found by mutating this instrument, not by reading it:**

- **Awk's `.*` is greedy.** `sub(/.*"type":"tool_use"/, "", line)` skips to the **last**
  occurrence on the line, so a parallel batch reports its last tool as its first. The reader
  splits on the marker instead.
- **A transcript being read while it is still being written can hand you half a line** — the
  marker present, the name not yet. Scoring it turns a real firing into a miss that nothing
  downstream can detect. Only complete JSON lines are read.
- **The first sole-catcher table this page was going to carry was evidence of nothing.** The
  harness that produced it could not call the function it was testing, so every fixture
  "caught" every mutation, identically, in every row. It was rebuilt with a baseline assertion
  — the unmutated reader must read all five fixtures correctly before any mutation runs — and
  that baseline is what caught it. **A mutation harness needs its own calibration, exactly like
  the thing it mutates.**

## Cost, and why it is never automatic

A full run is killed at the first batch: **~6 s** per run instead of ~70 s. The strict
definition and the cheap one turned out to be the same thing — stopping at the first batch is
all the strict reading needs.

Two things that must not be assumed:

- **Closing the pipe does not stop the runner.** An `awk` that exits on the first match left it
  going for 52 seconds. The kill is explicit, by PID.
- **macOS ships no `timeout` and no `gtimeout`.** The polling ceiling in `run` is the only
  thing that stops a hung run.

Even at 6 s it is model-backed and non-deterministic, so it **cannot** join `.githooks/pre-commit`
and it **cannot** join `lenses/run`, which is deterministic and finishes in under a second. It
runs when you ask it to.

## Adding a case

1. It must come from what the skill already declares. Mark it `quoted` or `derived` honestly.
2. Adding a case **changes the corpus hash**, so every earlier reading stops being comparable.
   Batch corpus changes, and treat the next run as a new baseline.
3. Never add a case and change a skill in the same breath — that is the one move that makes the
   delta meaningless.

## Adding a reader fixture

1. A transcript plus an `expected` file holding the exact reading.
2. **Mutate the reader and confirm your fixture goes red.** A fixture that has never been
   observed to fail is not a test.
3. **Shasum the file before and after the mutation.** A patch that silently matches nothing
   turns a passing result into evidence of nothing — that has now happened four times in this
   workspace, and not once was it caught by paying attention.
