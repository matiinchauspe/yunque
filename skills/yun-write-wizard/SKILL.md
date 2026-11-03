---
name: yun-write-wizard
description: Generate an interactive bash wizard that walks a human through steps only they can perform. Use when provisioning infrastructure, setting up credentials or CI secrets, walking an unfamiliar third-party dashboard, or running a one-off migration or cutover. Don't invoke this for steps the agent can perform itself.
---

# yun-write-wizard

A **wizard** is a bash script that walks a human, step by step, through a manual procedure that's tedious to do by hand and tedious to re-explain to an AI every time. It opens each URL, says exactly what to click and copy, captures the values, writes them where they belong (`.env`, the CI's secret store), confirms at every stage, and shows how many stages are left. It might configure third-party services, run a one-off migration, or move the project from one state to another.

The delightful UX is already solved by [template.sh](template.sh): stage-by-stage progress, confirmation gates, cross-platform URL opening (including WSL), hidden secret entry, idempotent `.env` upserts, `gh secret`/`gh variable` writes, and a closing summary. **Your job is only to scope the procedure and author its stages.** The library above the `STAGES` marker is identical in every wizard; that consistency is the point: never hand-edit it.

A wizard is ephemeral by default: built for one run, saved to `.yun/wizards/<project>/<slug>.sh` in the workspace (gitignored), deleted when the job's done. Only when the user asks for a repeatable setup path that should live in the repo: copy it into the repo's `scripts/`, commit it there, and link it from the README.

**No human in the session — a dispatched run (`skills/yun-implement/SKILL.md` "Dispatched runs")** — stop and report the manual step instead of authoring a wizard. A wizard waits on someone to click and paste; one left waiting is worse than naming the blocker.

## Process

### 1. Scope the procedure

Work out every manual step the human must take and every value that gets captured along the way. Read the repo first, don't ask cold:

- For setup: `.env`, `.env.example`, `.env.*`, `README`, `docker-compose*`, framework config, and the repo's CI config (`.github/workflows/*` for GitHub Actions — every `secrets.*` / `vars.*` reference there is a value the wizard must produce).
- For a migration or transition: the current state, the target state, and the irreversible actions between them.

Then show the user the ordered list of stages and the values each produces, and confirm: they may add, drop, or reorder.

**Done when:** every stage is named in order, and for each captured value you know (a) where the human gets it, (b) where it's written (`.env`, the CI's secret store, both, or nowhere; some stages are pure actions), and (c) whether it's secret (hidden entry) or public.

### 2. Map each stage's journey

For each stage, write the precise path a human follows: which URL to open, what to do there, where a value is shown, which variable it fills: e.g. "Dashboard → Developers → API keys → Reveal test key → copy". Where you don't actually know the current UI or the exact command, say so and ask the user or check the docs: never invent steps that may not exist.

**Done when:** every stage traces to concrete instructions a stranger could follow.

### 3. Author the wizard

Copy [template.sh](template.sh) to `.yun/wizards/<project>/<slug>.sh` (gitignored; create the directory if it doesn't exist). Replace the example stage with one `stage` per step, in dependency order. Use the library helpers: `stage`, `say`/`step`, `open_url`, `ask`/`ask_secret`, `write_env`, `set_secret`/`set_var`, `pause`/`confirm`. `set_secret`/`set_var` write GitHub Actions only: for any other CI, `open_url` its secret settings and `step` the human through setting each value there. Set `TOTAL_STAGES` to the number of stages you wrote.

Hold the bar the template sets: open the URL before asking for its value, use `ask_secret` for anything secret, `write_env` every persisted value, `set_secret` only the values CI actually needs, and `confirm` before any irreversible action. Each `stage` clears the screen so only the current step is visible: keep a stage to one focused task so nothing the human needs scrolls away. Don't touch the library above the marker.

### 4. Verify and hand off

- `bash -n <script>`; run `shellcheck` if available.
- `chmod +x <script>`.
- Don't run it end-to-end yourself: it opens browsers and blocks on human input. Trace it statically instead: every value from step 1 is captured and lands where step 1 said, and every `set_secret` name exactly matches a `secrets.*` reference in CI.
- Tell the user to run it from the root of the checkout the work runs in (`repos/<project>`, or its worktree under `.worktrees/<project>/<task>/`), since `ENV_FILE="${ENV_FILE:-.env}"` is relative to the working directory, and to call the script by its absolute path so one command works from either: `cd <checkout> && bash <workspace>/.yun/wizards/<project>/<slug>.sh` (in Claude Code, with the `!` prefix). If it's a repeatable setup path the user asks for, copy it into the repo's `scripts/`, commit it there, and link it from the README so the next person runs the script instead of asking an AI.

## Attribution

Adapted for this workspace from **Matt Pocock's `wizard`** (github.com/mattpocock/skills, MIT).
Changes: renamed to the `yun-<verb>-<noun>` harness convention; a wizard is written to
`.yun/wizards/<project>/<slug>.sh` in the workspace rather than the target repo's scratch or
`scripts/`, so the hand-off runs it by absolute path from the checkout the work runs in (the
repo or its worktree), where its relative `ENV_FILE` must resolve; the skill text names "the CI's secret store" rather than GitHub
by default, since the `gh secret`/`gh variable` helpers apply only where the repo's CI is
GitHub Actions, and any other CI gets a stage that walks the human through its own secret settings; a dispatched run with no human in the session is a stop, never a wizard left
waiting; and `template.sh`'s `ask_secret` gets one added, `# yunque:`-marked line that echoes the
captured secret's length, never its value. The four-step process, its "Done when" criteria, the
"never hand-edit the library" rule, and "ephemeral by default" are preserved verbatim; the
library above the `STAGES` marker, including the `gh secret`/`gh variable` helpers themselves,
is untouched.
