# Lenses — the harness's own check suite

A **lens** is one question asked of the whole corpus. Not "is this file good" — one
property, swept across every file, answered pass/fail.

This is a linter for the harness, not an eval. It is deterministic, needs no model,
and runs in under a second. Evals — does the harness actually change how an agent
behaves — are a different instrument for a different question, and they cannot be
trusted while the text they grade still contradicts its own index.

```
lenses/run                 # every lens: calibrate against fixtures, then measure the harness
lenses/run invocation-flag # just one
lenses/<lens> <root>       # one lens against an arbitrary tree
```

Exit: `0` clean · `1` findings in the harness · `2` a lens is uncalibrated, could not run,
arrived without its executable bit, or there were no lenses to run at all. **Anything but
`0` and `1` means nothing was measured, which is never a clean result.**

## Every lens carries fixtures, and that is the point

A lens with no fixtures is not trusted and `run` refuses to report its reading.

The reason is measured, not theoretical: a broken check does not disable a
verification — it *invents* one, and it arrives wearing the costume of a rigorous
finding, with file names attached. It happened on 2026-09-17, and again on 2026-09-20
when a fresh agent with no memory of the first rebuilt the same broken instrument from
scratch. Discipline caught both. Discipline does not scale.

A fixture is the **calibration block**: a planted case whose correct reading is
obvious by inspection. That is where the who-checks-the-checker recursion stops —
not in principle, but because a six-line fake skill can be verified by a human eye
and 4,701 lines of prose cannot.

**A fixture is only proven by mutation.** Green fixtures against a working lens prove
nothing. Break the lens on purpose and confirm the fixture goes red. The first version
of `clean-documents-the-flag` passed every run and still failed to catch a broken
lens — found only by mutating, and fixed by planting the missing form.

## The registry

| Lens | The question it asks | Fixtures | Caught in the wild |
| ---- | -------------------- | -------- | ------------------ |
| `invocation-flag` | Does each skill's invocation mode, read from **frontmatter only**, agree with the `AGENTS.md` index? Also: any skill missing from the index, any index row with no skill. | 4 | Twice, three days apart and by two agents who never met, an unanchored whole-file match reported `yun-write-skill` as user-invoked when its frontmatter is clean. |
| `structural-form` | Does every `SKILL.md` open with an H1 that is **its own name**? First H1 only, outside the frontmatter and outside code fences. | 4 | `yun-research` carried no H1 at all, and `yun-slice-plan`'s only heading was `# <NN> — <Ticket title>`, a form field from inside its own template. |
| `line-citations` | Does every line citation resolve — the cited file present in the corpus, and the cited line or **range** inside its length? Reports `DANGLING` / `AMBIGUOUS` / `OUT-OF-RANGE`. **It never checks that the line still SAYS what the citing sentence claims** — content drift is not mechanical, and a lens implying otherwise would be inventing a verification rather than performing one. | 5 | `plan-prompt.md:25` in `yun-build-plan/LIMITS.md` — a line number pointing into a file this workspace does not hold, missed by two hand sweeps whose regex did not match ranges. |
| `frontmatter-name` | Does every skill's frontmatter `name:` match its **directory name**? Frontmatter only; one layer of matching quotes is stripped, because `name: "yun-x"` is valid YAML and flagging it would invent a defect. Reports `MISMATCH` / `NO-NAME`. | 4 | **Nothing yet, and that is the point** — a drifted name does not fail, it makes the skill undiscoverable in silence while every other lens goes on reading `CLEAN`. Measured by planting `name: yun-reserch` in `yun-research`. |

`structural-form`'s three exclusions are each grounded in this corpus, not imagined, and
each has a fixture that a naive lens fails:

- **fences** — `yun-model-domain` carries `# Event-sourced orders` inside one. Fixture:
  `fenced-h1-only`, a skill whose *only* `#` line sits in a fence; a fence-blind lens
  calls it CLEAN.
- **the space in `^# `** — without it `##` and `###` match too. Fixture: `no-h1`, which
  has section headings and no H1; a `^#` lens reports WRONG-H1 instead of NO-H1, describing
  a true defect falsely.
- **first H1 only** — `yun-slice-plan` and `yun-write-spec` legitimately carry a second H1,
  the title of the document they *produce*. Fixture: `clean-template-second-h1`; a lens
  that checks "exactly one H1" flags both real skills.

`line-citations` carries two decisions worth knowing before trusting its reading:

- **It scans `skills/` and `AGENTS.md`, and deliberately NOT this file.** The registry quotes
  citations as worked examples, so scanning it would manufacture findings out of its own
  documentation. A check that reads the prose describing it reports defects that do not exist —
  that has happened twice in this workspace, and both times the false finding arrived with a
  file name attached and looked rigorous. Resolution, by contrast, searches the WHOLE corpus:
  a citation into `lenses/run` is as real as one into a `SKILL.md`.
- **A missing target is a FINDING, not a warning, even though some targets are external.** The
  lens cannot tell "cites the source template we do not hold" from "cites a harness file that was
  renamed" — and the second is the rot it exists to catch. Making the first silent would blind it
  to the second. So the practice is to repair an unverifiable line number into a file-level
  reference, per the running list's own rule: name the file, quote the sentence, never a line number.

**Six mutations, five fixtures, none redundant** — each fixture is the sole catcher of at least
one break:

| Mutation of the lens | Caught only by |
| -------------------- | -------------- |
| stops matching ranges (`:114-123` → `:114`) — **the bug both hand sweeps had** | `range-out-of-bounds` |
| stops skipping code fences | `fenced-citation-only` |
| resolves a target only WITH its extension, never without | `clean-range-and-single` (+2) |
| accepts a basename two files share | `ambiguous` |
| accepts a target absent from the corpus | `dangling` |
| off-by-one at the boundary (`-gt` → `-ge`) | `clean-range-and-single` |

**That last row is the whole reason this page insists on mutation.** The first
`clean-range-and-single` cited only an interior line and a range, and the off-by-one passed
**all five fixtures** — a green suite hiding a lens that rejects every citation to a file's
last line. Found by mutating, fixed by citing the last line on purpose. **A clean fixture
must exercise the boundary, not just the happy middle.**

`frontmatter-name` is the first lens here written **before** its defect appeared in the
corpus, and the failure's shape is the reason. A name that drifts from its directory does
not break loudly: the skill stops being discoverable, nothing reports it, and the other
three lenses stay green. Measured by planting `name: yun-reserch` in `yun-research` — the
suite read `CLEAN` while the skill was uninvocable.

**No skill body carries a `name:` line today**, so the whole-file read — the instrument bug
rebuilt from scratch four times in this workspace — would not fire here. It would sit latent
until someone documents the key in `yun-write-skill`, which is precisely where it belongs.
`clean-documents-the-name` plants all three prose shapes now, so the lens cannot acquire that
bug quietly later.

**Six mutations, four fixtures, none redundant:**

| Mutation of the lens | Caught only by |
| -------------------- | -------------- |
| loses the `^` anchor and matches `name:` mid-line | `clean-documents-the-name` |
| stops stripping quotes (`name: "yun-x"`) | `clean-quoted-name` |
| stops comparing the two names at all | `mismatch` |
| reads the whole file, first match | `no-name` |
| treats a missing name as clean | `no-name` |
| reads the whole file, every match | `clean-documents-the-name` (+1) |

**`no-name`'s planted key points at its own directory on purpose.** A whole-file lens reads
the body's `name: yun-anon`, finds it matching, and reports `CLEAN` — a *false* clean, which
is the silent direction and the one a green suite hides. A plant that disagreed would have
produced a loud `MISMATCH` and proved strictly less.

### Lenses already run by hand, not yet mechanised

From the traversals of 2026-09-13 and 2026-09-17. Listed so the set is a file rather
than something an agent has to remember: a traversal converges per LENS, and its
stopping rule never said so — so a run that "converged" had converged on the two
questions someone happened to remember.

| Lens | Question | Status |
| ---- | -------- | ------ |
| references | Does every cross-reference resolve? | ran clean by hand (131 refs); the `sd` pipeline that ran it collapsed its matches and reported one empty row — the instrument failed, not the corpus |
| branches | Does every branch of every decision tree terminate? | ran clean by hand |
| counters | Does every "N things" in prose match the count that follows? | ran clean by hand |
| consumers · attribution · duplication · prior-decisions · negation · completion-criteria | recorded in the 2026-09-17 traversal | ran by hand |

Mechanising one is worth it when it has caught something, has failed silently, or is
cheap. Four lenses ran clean on 2026-09-17 and only the cheapest found anything — the
other three were worth running to learn they were clean.

## What fires this suite

Two layers, with different jobs. Neither alone is enough, and the first is prose —
which is exactly why the second exists.

| Layer | Reaches | When | Tool-agnostic because |
| ----- | ------- | ---- | --------------------- |
| The closing assertion in `yun-write-skill` | harness text authored **through a skill** | in session, while a fix is still cheap | it is prose in a `SKILL.md` |
| `.githooks/pre-commit` | **everything else** — a line changed in `AGENTS.md` mid-conversation, an edit made by hand | at commit, against the **staged** tree | it is git |

The assertion is a closing one, never a precondition: the precondition form is
measured 8-for-8 broken across this harness, and mid-edit the suite is *expected* to
report — an index that does not yet list the skill being added is correct.

Activate the hook once per clone:

```
git config core.hooksPath .githooks
```

**Promotion rule.** The assertion lives in `yun-write-skill` and nowhere else,
because that is its only measured consumer. `yun-review-work` is the likely second.
**When a THIRD skill needs it, move it to a contract under `skills/_shared/` and
have all three reference that** — the six existing contracts each resolve a
capability with a ladder and a home, and this is a verification step, a different
shape. Two consumers do not justify one; promote on the third, measured, not before.

## Adding a lens

1. Write the question down here first. One question.
2. `lenses/<name>` — takes a root, prints findings, `0` clean / `1` findings / `2` cannot run.
   **`chmod +x` it.** `run` collects lenses by their executable bit, so a lens without one
   used to drop out of the suite with no message while the verdict still spoke for "every
   lens". It now reports `NOT EXECUTABLE` and refuses to read clean — but the bit is still
   yours to set.
3. `lenses/fixtures/<name>/<case>/` — a tree plus an `expected` file, one per condition
   the lens reports, **plus at least one case that must read CLEAN**. The clean case is
   the one that catches a lens gone greedy.
4. **Mutate the lens and confirm the fixtures go red.** Skip this and you have written a
   check that has never been observed to fail, which is not a check.
