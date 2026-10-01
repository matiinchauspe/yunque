# Conduct — once a skill fires, does it do what it says?

`evals/run` stops at the first tool call: it asks whether the right skill **fired**. A skill
can fire perfectly and then not do its job — the grill once answered *"Decisión 1 registrada"*
and wrote nothing. That is **conduct**, and it only shows across a whole run, over several
turns, in what actually landed on disk.

```
evals/conduct/run --gate0                     # the evaluator alone — free, no API calls
evals/conduct/run --calibrate                 # both gates, no measurement — ~$0.30
evals/conduct/run                             # every case, 3 runs each, harness = working tree
evals/conduct/run --harness main reservee-grill
evals/conduct/run --runs 5 --keep reservee-grill
```

Exit: `0` measured · `1` measured, and some invariant held **0 of N** · `2` **not measured**.
As in `evals/run`, there is no threshold between 0 and 1: 2/3 against 3/3 is not significant —
re-run with `--runs 5` when a fix lands there.

## What it judges, and what it never reads

Each turn is judged against **invariants the case declares**, from two sources only: the tool
calls in that turn's transcript, and the files that changed in the sandbox **during that turn**.
The agent's prose is never read and no model sits in the measuring path. Accepted cost: a false
claim about something the case did not declare goes unseen.

The vocabulary is **closed** — four predicates, one per line of `turn-N.expect`:

| Predicate | Holds when |
| --- | --- |
| `invoked <skill>` | the turn made a `Skill` call for `<skill>` |
| `wrote <glob>` | a file matching `<glob>` was created, modified **or deleted** |
| `added <glob> <regex>` | a file matching `<glob>` gained a line matching `<regex>` (ERE) |
| `not <predicate>` | the negation of one of the three — once, never nested |

Globs match paths relative to the sandbox root, **the whole path**: `**` crosses directories,
`*` and `?` stay inside one segment, `.` is literal. So `.yun/memory/reservee/*` does not match
a file one directory further down.

## The sandbox is a whole workspace root

`.yun/memory/` resolves from the workspace root and skills load from the agent's cwd, so each
run gets a temp dir holding the harness, the case's repo **cloned at its pinned commit**, and an
empty `.yun/`. The agent runs there with `--setting-sources project` (engram absent, so persist
falls to the diffable file floor) and `--permission-mode acceptEdits`, which confines `Write` and
`Edit` to that cwd — they are deliberately **not** in `--allowedTools`, which would grant them
everywhere. The sandbox is deleted afterwards; `--keep` leaves it.

## Comparable means the same harness hash

The report prints a git **tree hash over the harness only** — `skills/`, `AGENTS.md`,
`CLAUDE.md` and the `.claude/skills` / `.cursor/skills` symlinks — never over `evals/`, so
editing a case or this instrument never makes two readings of one harness look different. It is
content-addressed: a clean working tree and `--harness main` hash alike (verified).

## History, and the before/after it gives you for free

Every reading appends one row to `.yun/evals/conduct/<case>.tsv` — local, gitignored, never
committed: date, harness hash, case hash, runner, model, k, and each invariant's fraction. The
report then shows the most recent reading of the **same case** under a **different harness**,
beside the new one. A reading belongs to one runner and model on one machine; committed, it
would read as "the harness's number". Evidence that must travel goes in the PR, by hand.

## Calibration — two gates, and nothing is reported if one fails

| Gate | Asks | Costs |
| --- | --- | --- |
| **0 · the evaluator** | does `judge` read planted per-turn snapshots and transcripts correctly, and does the validator refuse what is outside the vocabulary? | nothing |
| **1 · the live control** | turn 1 writes `hola` to `.yun/memory/control/x.md`, turn 2 appends `chau` — `wrote` and `added hola` in turn 1 **only**, `added chau` in turn 2 **only** | ~$0.30 |

Gate 1 proves what gate 0 cannot: `--resume` continues the **same** session (checked by id, and
turn 2's prompt names no path), `.yun/` resolves **in the sandbox and not in the real
workspace** (checked against the real `.yun/memory/control/`), and the snapshots are taken at
the turn boundary. It runs through `run_case`, like any case — the same way `judge` is the one
function gate 0, gate 1 and the measurement all go through.

## The fixtures, and what each one alone catches

| Mutation of the evaluator | Caught by |
| --- | --- |
| a turn judged against snapshot 0 instead of the previous one | **`turn-boundary` alone** |
| `added` reads the whole file instead of the lines the turn added | `turn-boundary` |
| modifications unseen (only creations count) | `turn-boundary` · `unchanged-and-deleted` |
| deletions unseen | **`unchanged-and-deleted` alone** |
| `*` crosses `/` · `.` unescaped · glob unanchored | **`glob` alone** |
| reads only the last block of a parallel batch (awk's greedy `.*`) | **`invoked-reader` alone** |
| stops reading at the first batch (the `evals/run` reader, reused) | **`invoked-reader` alone** |
| any tool with a `skill` key counts as a Skill call | **`invoked-reader` alone** |
| `not` does not negate | every fixture carrying a `not` |
| the validator accepts anything · `not` nests | **`rejected` alone** |
| a verdict taken from a pipeline's exit status | `turn-boundary` · `unchanged-and-deleted` |

Fourteen mutations, all red, each patch verified applied by `shasum` before and after, against
a baseline in which the unmutated copy calibrates in the same place. Every fixture is the sole
catcher of something.

**What gate 0 does not cover, stated plainly:** the composition of turns in `run_case` — which
snapshot and which transcript each live turn is judged with. That is gate 1's job, and gate 1 is
what proved it.

**Two traps, found here, beyond the three in `evals/README.md`:**

- **Never decide on a pipeline's status under `pipefail`.** `diff | sed | grep -q` is false
  exactly when a line *was* added, because `diff` exits 1 on any difference; and `… | grep -q`
  goes false whenever `grep` closes the pipe early enough for the writer to die of `SIGPIPE`,
  which only happens once the input is big. Gate 0 caught the first on its first run. Every
  verdict now captures the text first and tests it with a here-string.
- **A mutation can apply and still not mutate what it names.** The first "stops at the first
  batch" mutant changed the file — the shasum moved — and survived, because its `exit` sat in a
  branch the first batch never reached. `shasum` proves a patch landed, not that it means what
  its label says; a surviving mutant is a question about the mutant before it is one about the
  fixtures.

## Adding a case

`cases/<name>/` holds `case` (optional: `repo <name> <commit>`, `#` comments allowed), then
`turn-N.prompt` and `turn-N.expect` for N = 1, 2, … Every case is checked before a cent is spent:
each prompt has its invariants, every invariant is in the vocabulary, the pinned commit exists in
`repos/<name>`.

Changing a case changes its **case hash**, and the before/after only pairs readings of the same
case hash. Never change a case and a skill in the same breath.

## Cost

A turn runs to its end: ~70 s and roughly $1 on a real case; the control is two short turns.
Model-backed and non-deterministic, so it never joins `lenses/run` or `.githooks/pre-commit`.
Gate 0 alone is deterministic and free.
