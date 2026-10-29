---
name: yun-write-agent-docs
description: Use when authoring or editing a document an agent reads in this workspace — a skill, a `skills/_shared/` contract, `AGENTS.md` or `skills/README.md` — deciding how it is invoked, what lives in it versus a linked file, naming it, or diagnosing why a skill misfires, bloats, or fires unreliably.
---

# yun-write-agent-docs

A document an agent reads exists to wrangle determinism out of a stochastic system. **Predictability** — the agent taking the same _process_ every run, not producing the same output — is the root virtue; every lever below serves it. The document can be a skill, a `skills/_shared/` contract, `AGENTS.md`, `skills/README.md`, or a file a skill links to. The packaging differs; the writing does not.

**Bold terms** are defined in [`GLOSSARY.md`](GLOSSARY.md); look them up there for the full meaning.

**When the document is a skill**, also read [`SKILL-MECHANICS.md`](SKILL-MECHANICS.md) for its frontmatter, the invocation choice, splitting by invocation, and router skills.

## In this workspace

This is a harness skill. Two workspace rules bind every skill you author here — do not restate them, read them at their source:

- **Naming:** follow the `yun-<verb>-<noun>` convention in `AGENTS.md`. The directory name must match the skill `name`.
- **Home:** harness skills live in `skills/`; they are always active when your agent opens from the workspace root. A repo's own skills are secondary.

**Before authoring or diagnosing, `recall` the harness's `friction` topic** (through `skills/_shared/memory-convention.md`) — sessions working *other* projects file there what the harness got wrong. It is the harness's only supply of measured pain.

**Close an edit by syncing the index.** `AGENTS.md`'s skill table and its `skills/_shared/` list, and `skills/README.md`, are a derived view of `skills/`. Sync them as the **last** edit, once the **traversal** has converged — a traversal keeps editing what the index describes. **The view is derived, so derive it: do not declare the index synced until `lenses/run` has read `CLEAN`, and quote its last line.** An assertion you can satisfy by saying you ran it is a precondition in disguise. Mid-edit the suite is *expected* to report — an index that does not yet list the skill you are adding is correct — so this is a closing assertion and never a gate on the work.

## Context pointers

A **context pointer** is a line held in the agent's context that names out-of-context material and encodes when to reach it. A skill's **description** is one; an `AGENTS.md` line naming a contract is the same object. Its _wording_, not its target, decides when the agent reaches the material, and how reliably. A pointer does two jobs — state what the material is, and list the **branches** that should trigger reaching it. Every word of an always-loaded pointer costs **context load** on every turn, so it earns even harder pruning than the body:

- **Front-load the leading word** — the pointer is where it does its triggering work.
- **One trigger per branch.** Synonyms that rename a single branch are **duplication** — "build features using TDD … asks for test-first development" is one branch written twice. Collapse them; keep only genuinely distinct branches.
- **Cut identity the body already carries.** Keep a description to triggers, plus any "when another skill needs…" reach clause; keep an `AGENTS.md` line to what the material is and when to reach it.

## Information hierarchy

A document is built from two content types — **steps** and **reference** — that mix freely: all steps, all reference, or both. The core decision is which to use and where each sits on the **information hierarchy**, a ladder ranked by how immediately the agent needs the material:

1. **In-file step** — an ordered action, the primary tier: what the agent does, in order. Each step ends on a **completion criterion**, the condition that tells the agent the work is done. Make it _checkable_ (can the agent tell done from not-done?) and, where it matters, _exhaustive_ ("every modified model accounted for", not "produce a change list") — a vague criterion invites **premature completion**.
2. **In-file reference** — a definition, rule, or fact, consulted on demand. Often a legitimately flat peer-set (every rule of a review on one rung) — a fine arrangement, not a smell. _This skill is all reference._
3. **External reference** — reference pushed out into a separate file, reached by a **context pointer**, loaded only when the pointer fires. (Spans _disclosed_ reference — a sibling file like `GLOSSARY.md`, still part of the skill — through fully **external reference** that lives outside the skill system and any document can point at.)

A demanding completion criterion drives thorough **legwork** — the digging the agent does within the work — whether the document has steps or not, since "every rule applied" binds flat reference just as "every step done" binds a sequence.

Push too little down and the top bloats; push too much and you hide material the agent actually needs. That tension is the whole decision.

**Progressive disclosure** is the move down the ladder — out of the main file into a linked one — so the top stays legible. Mechanics: a linked `.md` file beside it, named for what it holds (this skill discloses its full definitions to `GLOSSARY.md`, and its skill-only material to `SKILL-MECHANICS.md`). **When to pull the lever:** the length itself is the trigger — treat a single reference block past ~50 lines, or a `SKILL.md` past ~150–200 lines (~500 words for a frequently-loaded skill), as **sprawl** and disclose it; no other justification is needed. Some documents are used in more than one way, and each distinct way is a **branch** — different runs taking different paths through it. Branching is the cleanest disclosure test: inline what every branch needs, and push behind a pointer what only some branches reach. Disclosing moves antecedents behind that pointer and leaves every branch but the one that opens the file holding them, so close a disclosure with a **traversal**.

Where the ladder decides _how far down_ a piece sits, **co-location** decides _what sits beside it_ once there: keep a concept's definition, rules, and caveats under one heading rather than scattered, so reading one part brings its neighbours with it.

## When to split

**Granularity** is how finely you divide documents, and each cut spends one of the two loads, so split only when the cut earns it. Two cuts:

- **By sequence** — split a run of **steps** when the steps still ahead (a step's **post-completion steps**) tempt the agent to rush the one in front of it (**premature completion**). Keeping them out of view encourages the agent to do more **legwork** on the current task — but only across a real context boundary: a hand-off or a subagent dispatch.
- **By invocation** — a skill-only cut; see [`SKILL-MECHANICS.md`](SKILL-MECHANICS.md).

## Pruning

Keep each meaning in a **single source of truth**: one authoritative place, so changing the behaviour is a one-place edit. In this harness the contracts are the sharpest case: each `skills/_shared/` file is the source of truth for itself, so `AGENTS.md` and `skills/README.md` point at it rather than restate it.

The environment is a source of truth too — `package.json` scripts, config files, the directory layout, `--help` output — and a document that restates it is a **cache**. A cache earns its load only when the lookup is expensive: write down what the agent cannot find by looking (the unwritten convention, the reason behind a choice, the gotcha no config confesses), and leave the one-file, one-command lookups to the environment, where they cannot go stale.

Check every line for **relevance**: does it still bear on what the document does?

Then hunt **no-ops** sentence by sentence, not just line by line: run the no-op test on each sentence in isolation, and when one fails, delete the whole sentence rather than trim words from it. Be aggressive — most prose that fails should go, not be rewritten.

## Leading words

A **leading word** is a compact concept already living in the model's pretraining that the agent thinks with while running the document (e.g. _lesson_, _fog of war_, _tracer bullets_). Repeated throughout the text (though not necessarily — a strong leading word might only be needed once), it accumulates a distributed definition and anchors a whole region of behaviour in the fewest tokens, by recruiting priors the model already holds.

It serves predictability twice. In the body it anchors _execution_: the agent reaches for the same behaviour every time the word appears. In a pointer it anchors _invocation_: when the same word lives in your prompts, docs, and code, the agent links that shared language to the material and reaches it more reliably.

Hunt for opportunities to refactor documents to use leading words. A triad spelled out at three sites (**duplication**), a pointer spending a sentence to gesture at one idea — each is a passage begging to **collapse** into a single token. Examples include:

- "fast, deterministic, low-overhead" -> _tight_ — one quality restated across a phase — into a single pretrained word (a _tight_ loop).
- "a loop you believe in" -> _red_ — converts a fuzzy gate into a binary observable state (the loop goes _red_ on the bug, or it doesn't).

You win twice over: fewer tokens, _and_ a sharper hook for the agent to hang its thinking on. Assume every document is carrying restatements that leading words retire — go find them.

## Failure modes

Use these to diagnose issues the user may be having with a document.

- **Premature completion** — ending a step before it's genuinely done, attention slipping to _being done_. Defence, in order: sharpen the completion criterion first (cheap, local); only if it is irreducibly fuzzy _and_ you observe the rush, hide the post-completion steps by splitting (the sequence cut).
- **Duplication** — the same meaning in more than one place. Costs maintenance and tokens, and inflates a meaning's prominence on the ladder past its real rank.
- **Sediment** — stale layers that settle because adding feels safe and removing feels risky. The default fate of any document without a pruning discipline.
- **Sprawl** — a document simply too long, even when every line is live and unique. Hurts readability and maintainability and wastes tokens. The cure is the ladder: disclose **reference** behind pointers, and split by **branch** or sequence so each path carries only what it needs.
- **No-op** — a line the model already obeys by default, so you pay load to say nothing. The test: does it change behaviour versus the default? A weak leading word (_be thorough_ when the agent is already thorough-ish) is a no-op; the fix is a stronger word (_relentless_), not a different technique.
- **Negation** — steering by prohibition backfires: _don't think of an elephant_ names the elephant and makes it more available, not less. Prompt the **positive** — state the target behaviour so the banned one is never spoken; keep a prohibition only as a hard guardrail you can't phrase positively, and even then pair it with what to do instead.
- **Dead branch** — a **branch** that cannot be completed as written: it closes on a **completion criterion** only a different branch can satisfy, or it reads a reference that resolves nowhere along this path. The defect lives on a path, not in a passage — every passage is correct read alone, so re-reading the edited region cannot surface it, and neither can a reviewer of the version before the edit. A skill is a program with no test suite, and reading one is not running it. Defence: a **traversal**. The trigger is the document's _shape_, not your edit: traverse every branch of any document you touch that carries more than one, including the branches no edit went near — a defect born in an earlier edit sits dormant on an untravelled path until someone walks it. A traversal is not one-shot: every repair made during one invalidates it, so re-run until it converges — a pass that finds nothing and changes nothing. **Show it: write each pass out as a table of the branches walked, and declare convergence only on a pass whose table reads clean** — a traversal walked in your head is an act with no output, and one reported converged on its second pass found three more defects once written out. A linear document has no path to walk, so traversing it is a **no-op**.

## Attribution

Adapted for this workspace from **Matt Pocock's `writing-for-agents`** (github.com/mattpocock/skills, MIT), first ported from its earlier form, `writing-great-skills`. Changes: renamed to the `yun-<verb>-<noun>` harness convention; the harness bindings added — the `friction` recall, the index sync closed on `lenses/run`, and the **dead branch** / **traversal** failure mode, which upstream does not carry; the terms kept as a `GLOSSARY.md` dictionary where upstream now writes them as prose; workspace mechanics deferred to `AGENTS.md`; and the OpenAI agent wiring and aihero.dev/`ask-matt` ecosystem references dropped. The scope over any agent-read document, the **cache** rule, and the `SKILL-MECHANICS.md` split follow upstream.
