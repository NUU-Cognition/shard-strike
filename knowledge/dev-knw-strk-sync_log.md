---
description: "The sync log of Strike — where it is, how to read it with flint foundation logs, where to look first for each sync fault, the privacy rule, and the event names"
orbh-sessions:
  - "[[6b4b8648-dc9c-4d81-be42-d21da49a70aa]]"
---

# Knowledge: Sync Log

Strike writes each sync event to the event log of its Foundation registration ((Task) 311). Read this log first when a Strike sync fault is reported. It shows what happened, in order, with no reproduction. Both sync paths write to it: the replica path and the command path ([[knw-strk-command_sync]]).

## Where the Log Is

| | |
|---|---|
| File | `<NUU home>/foundation/logs/<registration ID>/events.jsonl`. At 10 MiB the file moves to `events.1.jsonl` and a new file starts. |
| The TUI | Registers as `strike/tui` right after the instance lock, so the log also holds the sync mode choice or the refusal at start. |
| A command line run | `strike apply`, `finish`, `start`, `reset`, and `session finish` with no running Strike register as `strike/cli` and write their own log. `strike target show` writes no log. |
| After an exit | The log stays after the process exits. `flint foundation prune --apply` removes exited logs older than 7 days, or the oldest exited logs above 200 MiB in total. |

Each line is one JSON object: `ts`, `seq`, `level`, `event`, `service`, `role`, `pid`, `id` (the registration ID), and `data`. Each file starts with a `log.open` line that holds the identity of the process. A clean end writes `log.close`. A log with no `log.close` line ended with a crash or a kill. A gap in `seq` means lost lines.

## How to Read It

```bash
flint foundation logs --service strike --role tui --event sync --since 30m   # the TUI, the last 30 minutes
flint foundation logs --service strike --list                                # every Strike log: live or exited
flint foundation logs <registration ID> --level warn                         # one log, warnings and errors only
flint foundation logs --service strike --role cli                            # the newest command line run
flint foundation logs --service strike --event sync.command --json           # one event family, as JSON
```

- `--service strike` alone reads the newest Strike log, and that can be a command line run. Add `--role tui` or `--role cli`.
- `--event` matches whole dot-separated words: `sync.replica` matches `sync.replica.pass`, not `sync.replicas`. Repeat the flag for more than one prefix.
- `--limit` keeps the last lines (default 200). `--limit 0` shows all lines.
- The level is `info`. `STRIKE_SYNC_LOG_LEVEL=debug strike` adds one line for each send, each arrival, each checkpoint, and each cached token. Ask the operator before you restart a Strike that runs.

## Where to Look First

| Fault | Read |
|---|---|
| The socket drops, Strike shows offline | `sync.connection.state`: each state and `msInPrevious`. `sync.auth.state`: whether the server accepted the token. |
| The token expires under the socket ((Task) 305) | `sync.auth.token` with `forced: true`. `origin: stored` or a large `ageMs` means that the mint failed and the client got an old token. `sync.auth.error` gives the code of a failed fetch. |
| A command does not apply (command path) | Follow its `commandId` through `sync.command.submit`, `sync.command.send`, and then `applied`, `rejected` (with the revision conflicts), `unknown`, or `refused`. |
| A write does nothing | `sync.command.refused` with `stage: gate` gives the presence state or the gate code that refused it. |
| A row does not reach a peer (replica path) | `sync.replica.pass` on the writer, then `sync.replica.apply` on the peer. `sync.replica.conflict` names each row that the server refused. `sync.replica.adopt` names each row that took the server copy. |
| The badge shows `SYNC FAILED` | `sync.failure`: each entry of the failure sink, with its phase, unit, and code. A merged repeat writes a line too. |
| Strike refuses to start | `sync.mode.refused`: the orgs of the mix and the protocol of each. |

## The Privacy Rule

- The log holds ids (target id, replica id, command id, unit key `sub:<id>` or `backlog:<id>`, org slug), kinds, codes, revisions, counts, and durations in ms.
- The log never holds a title, a body, a label, a `mountLabel`, a scope name, a person name, or a token. A `message` field is `null` when the error text has a quote character, because Strike quotes each title in a message. Use `strike target show <id>` to find the title of a row.

## The Event Names

| Family | Events |
|---|---|
| Start and gate | `sync.mode.selected`, `sync.mode.refused`, `sync.mode.changed`, `sync.gate.closed`, `sync.gate.opened` |
| Failure sink | `sync.failure` |
| Connection and auth | `sync.connection.state`, `sync.presence.state`, `sync.auth.token`, `sync.auth.error`, `sync.auth.state`, `sync.auth.changed`, `sync.auth.cleared`, `sync.mutation.timeout` |
| Command path | `sync.engine.open`, `sync.command.submit`, `sync.command.send`, `sync.command.applied`, `sync.command.rejected`, `sync.command.unknown`, `sync.command.refused`, `sync.command.retry`, `sync.command.pump`, `sync.draft.resend`, `sync.draft.discard`, `sync.snapshot.received`, `sync.subscription.error`, `sync.file.unreadable` |
| Replica path | `sync.replica.skip`, `sync.replica.pass`, `sync.replica.conflict`, `sync.replica.tombstone`, `sync.replica.claim`, `sync.replica.checkpoint`, `sync.replica.snapshot`, `sync.replica.apply`, `sync.replica.adopt`, `sync.replica.resubscribe` |
| Command line run | `sync.cli.start`, `sync.cli.end` |

## Source

The data fields and the level of each event are in the "Sync log" section of `apps/strike-tui/README.md` in the Main repository. `apps/strike-tui/src/sync-log.ts` owns the names, the data shapes, and the levels. The Foundation side (the writer, the `logs` command, and prune) is in `apps/foundation/docs/DESIGN.md` sections 4 and 6.
