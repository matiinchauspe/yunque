# Control convention

Shared contract for every `yun-*` skill or contract that tells a session-observing capability who
the running session is. A skill declares **intent** — `announce` — and this file maps it to
whatever capability the session has. The labels let the human see, from outside, which session is
working which ticket of which project.

## Rules

1. **The capability declares itself; the harness finds it by shape.** A skill says *"announce
   labels"*, this file says what shape answers it, and the capability names its own command to the
   session. Nothing in the harness names it.
2. **No capability is not an error.** The announce is skipped and the run goes on exactly as it
   would without it.
3. **There is no file floor.** The capability is the record. A Yunque-side ledger of labels would be
   a second source of truth beside it.

## Mechanism

```
announce(labels):
  1. A session-observing capability has declared an announce command to this session
     → run it with the labels as `<key>=<value>` pairs, as its OWN shell command.
  2. Else → skip.
```

- **Capability** = any session observer that declares, in the session's own context, a command
  that labels the session issuing it — a line its hook injects at session or subagent start, or an
  instruction it loads, naming that shell command. The convention does not know or care which one
  it is.
- **The declaration is the only proof.** A capability installed on the machine but silent in this
  session — a runtime it has no hook for — is absent here, so the announce skips.
- **The announce is captured, not executed.** The capability reads the command before it runs and
  labels the session that issued it. A subagent's announce labels the subagent, which is why the
  run itself announces, never the caller that dispatched it. Only a whole command is captured, so
  chaining or piping hides the announce.
- **One token per pair.** Values are split on whitespace, so a value never contains a space.
- **`project` is the grouping key:** the capability groups sessions by it. Any other key is free
  text, shown as given.
- **Labels accumulate per session.** A later announce adds or overrides only the keys it carries,
  so a closing `verdict` leaves `ticket` and `project` standing. Every exit announces, a re-entered
  loop's included, so the label always reads the run's current state.

## Scope resolution

`project` resolves exactly as in `artifact-convention.md`. The `ticket` value is the ticket's id in
the store that holds it: the tracker's own id where there is one (`RES-131`), else the
`<feature>/<NN>` of its file path (`checkout-flow/03`).

## Ownership

Control owns **announcing**, which tells the capability who a session is. It does NOT own what
the capability does with labels (grouping, notifications, a policy), nor any state about the run:
that belongs to whatever drives it, and the branch it leaves is the record.
