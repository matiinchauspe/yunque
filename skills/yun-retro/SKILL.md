---
name: yun-retro
description: Conduct a retrospective on a coding session — candidate improvements to the harness and the target repo, by severity.
disable-model-invocation: true
argument-hint: "Which session? (default: this one)"
---

# yun-retro

The user has asked for a **retrospective**. You are suggesting improvements to the coding agent's
**environment** to improve future runs. Here the environment has two **layers**: the **harness**
(`skills/`, the `skills/_shared/` contracts, `AGENTS.md`, `lenses/`) and the **target repo** the
session worked (its checks, its agent instructions, its conventions doc).

## Steps

1. Load `yun-write-skill` for the writing style guide. Done when it is loaded — every candidate
   that touches steering text is judged by its rules.

2. Read the primary sources for the session the user specifies. This may mean searching the session
   transcripts your agent keeps on this machine. If the user doesn't specify a session, default to
   the current one. Done when you have read the session itself, not a summary or a baton about it.
   Where the named session cannot be found, name what you searched and ask which one was meant.

3. Look for candidates for improvement in these categories, across both layers — or the harness
   alone, when the session worked no repo. Done when every category has been weighed against the
   session on every layer it touched; a category with no candidate is a reading, not a skip.

- **Navigation**: how easy was it for the agent to find the right files? Are there hidden
  dependencies between files? Would a **navigation pointer** make it easier? _Use when_ the session
  took a long time to find a piece of information.
- **Automated checks**: are there automated checks that could catch errors the agent made? Linting,
  typing, tests, filesystem linters? Read each layer's own check command first — the repo's
  `package.json`/build-tool `lint`/`check` scripts and its CI workflow; the harness's `lenses/run`
  and `.githooks/pre-commit` — so a check that already exists but sits unwired or silently broken is
  the finding, not a reinvention. A repo with no **guardrail** (no pre-commit hook and no CI job
  running its lint/typecheck/test command) is itself a finding: an un-linted repo is a standing
  missed opportunity, not a neutral default. _Use when_ the agent made a mistake an automated check
  could have caught, or the repo has no guardrail at all.
- **Coding standards**: should the **reviewer agent** be given a new rule to enforce? Should an
  existing rule be removed or clarified? Classify the violation first: a **mechanical** one (a fixed
  syntactic pattern, a banned API, an import shape, a file-location rule) gets a deterministic
  check, full stop — in the repo, a custom rule in its own linter, a new pre-commit hook, or a new CI
  job, whichever its language and existing guardrail make cheapest; in the harness, a lens. Default
  to building the check over writing the rule. Reserve the repo's conventions doc for genuine
  **judgement calls** (cross-file consistency, "matches the surrounding style," anything no
  guardrail could ever substitute for); in the harness, a judgement call is a change to the skill
  or contract text itself. _Use when_ the reviewer agent failed to catch a mistake.
- **Global AGENTS.md**: are there any steering instructions that should be moved to coding standards
  (or automated checks) instead? _Use when_ the AGENTS.md file is particularly large — in the repo,
  in the harness, OR in the user's global scope.
- **Tool economy**: did the agent make expensive tool calls that could be streamlined? Is there any
  custom tooling (CLIs, MCPs) that is particularly token-inefficient? _Use when_ the agent made an
  expensive tool call.
- **No-ops**: look for instructions in steering files — skills and contracts included — that don't
  modify the agent's behavior. _Use when_ the steering files are large and unwieldy.
- **Information access**: look for opportunities to increase the agent's access to information.
  Teeing dev server logs, readonly access to third-party services. _Use when_ a crucial piece of
  information was not available to the agent.

4. Present these candidates to the user, in order of severity. Each one names its layer, its
   category, and the moment in the session that shows it. Done when every candidate is on the table
   — or, where step 3 found none, the categories it weighed are, with what each one read.

5. File only what the user accepts. A harness candidate goes to the `friction` topic through
   `skills/_shared/memory-convention.md` — recall it and write it back whole with yours added, never
   a note alone. A repo candidate is handed back as the change a person would make; recording it in
   the repo is that person's deliberate act, per `skills/_shared/practice-convention.md`, rule 4.
   Done when every accepted candidate is filed or handed back, and you have stated which went where.

## Reference

### Implementation vs Review

Remember that all work goes through two stages: implementation and review. The implementation agent
has the most **context pressure**. They are responsible for exploration, writing code, and
debugging failures — here, `yun-implement`'s build and `yun-tdd`.

The review agent has the least context pressure — it receives a diff, so no exploration needed. It
often does not need to write code or debug — here, `yun-implement`'s review stage and
`yun-review-work`.

This means that the review agent should be responsible for imposing coding standards, not the
implementation agent.

### Files

In the target repo:

- `CLAUDE.md`/`AGENTS.md`: these files are pushed to the context window of any agent working in this
  repo. They should be used incredibly sparingly, usually only for **navigation pointers** to other
  files.
- The conventions doc (Matt's `CODING_STANDARDS.md`; the repo may name its own — find it by
  property, as `practice-convention.md` does): read during review, not implementation. Add
  **navigation pointers** to docs folders if it gets more than 1,000 lines long.
- Docs: use docs as reference files, pointed to by other files. Look for existing docs before
  writing new ones.

In the harness:

- `AGENTS.md`: pushed to every agent opened from the workspace root. The same sparing rule holds.
- Skills and `skills/_shared/` contracts: a model-invoked skill's description goes into the agent's
  context window; a contract is read by the skills that name it. Follow the advice in
  `yun-write-skill`.
- `lenses/`: the harness's mechanical checks — where a mechanical harness candidate lands.

## Attribution

Adapted for this workspace from **Matt Pocock's `retro`** (github.com/mattpocock/skills, MIT).
Changes: the environment is read on two layers, the harness and the target repo; `yun-write-skill`
replaces `writing-for-agents`; the reviewer agent is `yun-implement`'s review stage and
`yun-review-work`, and `CODING_STANDARDS.md` is whatever conventions doc the repo keeps; a fifth step
files the candidates the user accepts — harness ones to `friction`, repo ones handed to a person,
since the harness never writes into the repo; the OpenAI agent wiring dropped.
