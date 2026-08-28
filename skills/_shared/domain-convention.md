# Domain convention

Shared contract for every `yun-*` skill that captures or consults a project's **domain
model** — its **ubiquitous language** and the decisions behind its shape. A skill declares
**intent** — `capture` or `consult` — and this file maps that intent to where the model
lives and how it degrades.

The model is two artifact kinds, and only two:

- a **glossary** — the ubiquitous language: each entry a term, a tight definition of what
  it *is*, and the synonyms it rejects. A glossary and nothing else.
- **decisions** — architecture decision records (ADRs): one per decision, why the shape is
  the way it is. Recorded sparingly, only for choices worth remembering.

`yun-model-domain` owns what those say; this convention owns where they land and how a reader
reaches them.

## Rules

1. **The model is a project deliverable, not harness scratch.** Unlike memory digests or
   tracked-work tickets, it lives committed in the target repo, versioned with the code it
   describes — the ubiquitous language travels with the code. It only leaves the repo when
   there is no repo to hold it.
2. **Scoped to a project** — one model per project, never global, never cross-repo.
3. **Capture always writes; consult degrades silently.** Capturing a term or a decision is a
   deliverable, so it always writes to the working tree and never skips — though the files
   commit only when the user asks (see *Mechanism*), never on their own. Consulting a model
   that isn't there is not an error: the reader proceeds silently — it does not flag the
   absence or suggest creating docs upfront, because the model is born lazily, the moment a
   term or decision actually resolves.

## Mechanism

The agent resolves intent to the first mechanism that fits, in order:

```
capture(entry, project):        # entry is a resolved term, or a decision
  The two homes are mutually exclusive — the work either targets a repo or it doesn't:
  1. project is a synced repo (project = <name>) → write in that repo:
       - a term     → the glossary at the repo root (CONTEXT.md)
       - a decision → docs/adr/<NNNN>-<slug>.md
  2. project is not tied to a repo (project = _workspace) → write the SAME layout under
       .yun/domain/<project>/ (workspace root, gitignored): CONTEXT.md + docs/adr/<NNNN>-<slug>.md.

  In both homes: a decision is a new file, numbered by scanning the active home's docs/adr/
  for the highest existing NNNN-prefixed file and incrementing by one — starting at 0001 when
  the directory is empty or absent — zero-padded to four digits.
  "Write" means land the file in the working tree — capture never skips — but the files
  commit with the branch only when the user asks; capture never commits on its own.

consult(area, project):  read the glossary for vocabulary, and the ADRs touching the area.
  1. capture's home for this project holds the files → read them.
  2. Absent → proceed silently. The model is created lazily; its absence is not a gap to fill.
```

There is no tool-capability tier: a domain model *is* committed versioned files — a human reads
`CONTEXT.md` in the repo — so unlike `memory-convention.md` and `artifact-convention.md`
there is nothing to abstract a tool over.

## Using the model

- **Vocabulary** — when your output names a domain concept (a test name, an interface, an
  issue title, a proposal), use the term as the glossary defines it; don't drift to a synonym
  it rejects. A concept the glossary is missing is a signal to capture it, not to name it ad hoc.
- **Decisions** — respect the ADRs in the area you touch. If your work contradicts one,
  surface it — *"contradicts ADR-0007, but worth reopening because…"* — rather than silently
  overriding.

## Scope resolution

`project` resolves exactly as in `memory-convention.md`. The floor is **single-context**: one
root glossary per project. A monorepo of several bounded contexts — one glossary per context
under its own directory, indexed by a root map — is a documented extension, not part of this
floor.
