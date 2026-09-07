# Practice convention

Shared contract for every `yun-*` skill that writes code into a target repo. A skill declares
**intent** — `conform` — and this file maps it to what gets read, how far it rules, and how it
degrades.

A repo's practice comes in two halves, and they are not read the same way:

- **declared** — what the repo says about itself: a conventions document, the linter and formatter
  config, its own agent instructions. Authoritative, cheap, and absent from most repos.
- **inferred** — what the repo does without saying so: naming, layering, error handling, test
  style. Present wherever code is, and ambiguous wherever the repo disagrees with itself.

The declared half is a **record**, written on purpose. The inferred half is a **mirror** of the
code as it stands today. That difference decides where each one lives and how far it rules.

## Rules

1. **Declared beats inferred** — the record outranks the mirror. A repo mid-migration declares the
   practice it is moving to while most of its code still shows the one it is leaving; conform
   follows the declaration.
2. **Practice governs form; the harness governs discipline.** The repo decides what the work looks
   like — what a test is called, where the file goes, which framework, how an error is shaped. The
   harness decides what is non-negotiable — whether the test is written at all, whether a
   done-signal fires, whether the work is reviewed, how it is committed. Conform to the form; a
   defect in the repo's practice is form you match, never discipline you drop.
3. **The inferred half is a by-product.** It comes off the files the work already opens — the one
   being changed and its neighbours — costing the run nothing beyond reads it was making anyway.
   Only where those yielded nothing does a single **targeted** pass follow, aimed at the areas the
   history keeps returning to and delegated, so the code lands in another context and just the
   digest comes back. A profile built from the whole tree is the read this contract does not make —
   not in the run's context, and not in a delegated one either. A thin digest is the expected
   first result: it sharpens as later runs open more of the repo.
4. **The contract reads the repo and never writes to it.** Recording a practice *in* the repo is a
   deliberate act with a person behind it — an ADR through `domain-convention.md`'s `capture`, or
   the repo's own conventions document. Conform supplies neither.
5. **Conform degrades silently.** A repo that declares nothing, with no code to read yet, is not an
   error and not a gap to announce. Write in the practice you have and move on.

## Mechanism

```
conform(area, project):  the practice governing this run — form only, per rule 2.
  Idempotent within a run: a second call from the same context reuses what the first
  one read. A build that reaches here again through its test loop pays nothing twice.

  1. Read what the repo DECLARES about how the work is done. Find it by PROPERTY, never by
     filename — projects name these differently and many keep them nested:
       - the toolchain's own configuration, linter and formatter. Nothing is guessed here:
         the package manifest names the commands and each tool resolves its own config.
         Mechanically enforced, so the most reliable declaration a repo carries — it is
         what the repo does, not what it meant to do.
       - the repo's agent instructions, wherever it keeps them, read as EVIDENCE about
         form. A source here, never a rule set — rule 2 caps what they reach, so a line
         of theirs claiming discipline lands nowhere.
       - prose that is PRESCRIPTIVE about the work — where components go, how an error is
         shaped — as against prose describing what the system does, which is domain and
         belongs to `consult`. Reach it through the repo's own index, its README and what
         that points at; one pass following pointers, never a crawl of the doc tree.
     ADRs are not read here — they belong to domain-convention.md's `consult`. A practice
     ADR read there outranks the digest, so a caller that skips `consult` reads them itself
     or loses that precedence.

  2. `recall` the digest (memory-convention.md) under the topic `repo-practice`.

  3. Digest absent → infer the facets from the code already open, and `persist` only what the
     inference actually yielded, under that same topic. Nothing open to infer from → rule 3's
     one targeted delegated pass, or, where that is out of reach, leave the topic untouched so
     the next run infers again: a digest persisted empty comes back on every later recall as a
     real answer, and holds the mirror blank for every run after it.

  4. Digest present → re-reflect it against the code already open: correct a facet that code
     CONTRADICTS, and add a facet the digest LACKS that the code now settles — the second is
     how a thin digest sharpens, per rule 3. `persist` only where something changed, leaving
     the facets this run saw nothing about as they are. One file is noise either way: the bar
     stays the one every facet is admitted under. Same by-product read as step 3 — it opens no
     file the run was not opening anyway.

  5. Neither half yields anything → proceed silently.
```

## The digest

One document per project, under the fixed topic `repo-practice`, holding the facets the inferred
half covers: **naming**, **layering**, **error handling** and **test style**.

Fixed and single because every consumer reads every facet at the same point in its flow. A recall
then has one address to try — it either hits or is genuinely empty, never empty merely because the
topic was spelled another way.

**A facet earns an entry only where the repo is consistent about it.** One recalls on every run
from here on, so a facet the repo disagrees with itself about earns nothing: an ambiguous rule
steers nobody and every later run pays to carry it. Short enough to be free is the bar.

**The newest write is the digest.** Step 4 writes over a topic that already holds a version, and
the ladder's tiers differ on what that does — one replaces, another accumulates. Where it
accumulates, the latest entry is the document and what sits behind it is history a recall reads
past. Persist the digest whole, never a patch against an earlier one, so the newest entry stands
alone.

**A mirror is re-reflected, never filed.** Writing the digest into the repo as a declaration would
leave the next run reading the harness's own inference back as though the team had declared it —
and rule 1 would then rank that guess above the code that falsifies it, permanently. So it stays a
mirror: step 4 corrects a facet the moment a run opens code contradicting it, and a practice changed
on purpose needs no correction here at all — that lands in the declared half, which step 1 re-reads
on every run and rule 1 already ranks above the mirror. What gets written down instead is the
*decision* to change, through rule 4's deliberate act: the only path from inferred to declared runs
through a person.

## Scope resolution

`project` resolves exactly as in `memory-convention.md`. Work not tied to a repo (`_workspace`) has
no repo practice to read, so conform degrades by rule 5 — the harness's own practice is
`yun-write-skill`'s subject, not this contract's.
