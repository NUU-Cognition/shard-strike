---
description: "The command sync of Strike — the model, the files, the badge, the keys, the states, and the limits. It is in effect only when the organisation has the sync protocol commands."
orbh-sessions:
  - "[[80188125-ac0b-4a4d-a942-f7f3ff1f3da4]]"
---

# Knowledge: Command Sync

Command sync is the second sync path of Strike ((Task) 300). Operations turns it on for one organization at a time. The query `strikeSync:mode` gives the protocol of an organization: `replica` (the old replication) or `commands`.

### The Model

- Operations is the authority for each shared fact. The files of Strike are a cache that Strike can build again.
- Each gesture is one command. The command is one transaction on the server. It applies completely or not at all.
- The engine is `SyncEngine` (`src/sync/engine.ts`). It implements `StrikeClient`, so the TUI reads the same rows as before.
- A verb runs on a copy of the old engine in memory. The difference of the rows becomes one command (`src/sync/plan.ts`).
- A gesture of more than one verb is one command. The undo journal (`CommandUndoManager` in `src/undo.ts`) runs `group` and `atomicGroup` as one engine gesture. A verb inside an open gesture joins it.
- Strike writes the command to its journal (with fsync) before it sends it. The row shows the change at once (the overlay). The change is saved when the server applies it.
- A timeout means "result unknown". Strike sends the same command again with the same command id. The server applies a command id one time only.
- A command carries the revisions that the operator saw. When another writer changed a row first, the server refuses the command. The row shows the server value, and the edit stays as a draft.
- One Strike process uses one path. At start Strike reads the mode of each organization that it holds a unit of (in the old store and in the held units of the engine). With no held unit, the default organization counts. All on `commands`: the command path. All on `replica`: the old path. A mix: Strike refuses to start and names each organization with its mode.
- When Strike cannot read the mode (offline, signed out, or a server with no `strikeSync:mode`), the last mode that this machine read counts. With no record, the organization is on `replica`. A server with no `strikeSync:mode` answers at once with an error; offline, the start of the TUI and a command line write with no running Strike wait up to 4 s for the answer.
- On the command path Strike constructs no `ReactiveTiedClient`, no replicator, and no applier. It calls only `strikeSync:*` functions. It never writes the old store.

### Units, Ids, and Moves

- A unit is a subsection or a project backlog. Its key is `sub:<id>` or `backlog:<id>`. The key is also the id of its anchor row and of its scope.
- A pull holds the unit and its dependency subsections. It sends no command. The snapshot of the unit brings the rows.
- A target id is a UUID, or `task:<taskId>` for a task that Operations created. The short reference (`target-304`) is an alias in the alias table. A command line, a session prompt, and an agent session name a row by the alias.
- An old local id of a tab, a session binding, the doing arrangement, the fights file, or a command line resolves through the alias table. Old scope ids (`scope-12`) of the old store become aliases of unit keys at start. New entries use target ids and unit keys.
- A target can have more than one alias. When the migration makes one target from two local ids of a device, both old ids resolve. Strike shows the first alias.
- A move is one command: `pick` (`a p`, `:pick`), a scope move (`W t`, `W b`), and the place picker (one `MoveTargets`). A move of a tied row moves its task in Operations in the same transaction. There is no canon tail.
- The first scope of a row is its placement. A new first scope moves the placement (`MoveTargets`). When the row keeps its old placement unit, that unit comes back as a reference (`SetMembership`) in the same command.
- `P` publishes a row as a task in the unit of its placement (`PublishTarget`). Promote is `PromoteTarget`. Tie (`m`, the tie overlay) and untie are `TieTarget` and `UntieTarget`. The tie overlay searches the tasks of Operations, as on the old path.
- The Operations page sends task edits, places, reorders, removes, new tasks, promotions, and due dates as commands. Edits of projects, deliverables, and subsections (create, delete, order, dependencies, priority, title), and writes of the master backlog, keep the existing mutations.

### Files

All files are beside the other stores of the data mode (`.dev` names on dev data). Do not edit them by hand.

| File | Holds | When it does not load |
|---|---|---|
| `strike-sync-journal.json` | Each command with no result, each applied command until a snapshot shows it, and each rejected draft | Copied aside; Strike starts with an empty journal and reports it |
| `strike-sync-cache.json` | The snapshots of the held units | Deleted; the snapshots build it again |
| `strike-sync-aliases.json` | `aliases`: the alias that Strike shows for each target id. `extra` (optional): each other alias, as alias to target id | Copied aside |
| `strike-sync-scopes.json` | The held units, their names and order | Copied aside |
| `strike-sync-doing.json` | The order of the doing rows | Copied aside |
| `strike-sync-mode.json` | The last mode that this machine read for each organization | Ignored |

The tabs file, the session bindings (`strike-orbh.json`), the people cache, and the fights file stay where they are.

### The Badge and the Sync Panel

- The badge is `SYNCED`, `SYNC PENDING n`, or `SYNC FAILED n`, as on the old path.
- Pending: the commands with no result yet. Failed: the rejected drafts and the entries of the failure log.
- `W f` or `:sync` opens the sync panel. A failure of a command has the phase `command`; a failure of a read of a unit has the phase `snapshot`.
- The panel shows each rejected draft with its row, the reason of the server, and the current value on the server.

### Keys

| Key | Where | Action |
|---|---|---|
| `u` | Anywhere | Undo: send the inverse command with the revisions of the result. A change by another writer refuses it: "Undo refused: another writer changed …" |
| `ctrl-r` | Anywhere | Redo: send the command again with the revisions of the undo. A change by another writer refuses it. |
| `J` / `K` | Sync panel | Select a rejected draft |
| `s` | Sync panel | Send the selected draft again, on top of the current rows |
| `x` | Sync panel | Discard the selected draft; the row keeps the server value |
| `o`, then `X` | Agent menu | Release the doing claim of another device, after a confirm (`y`). The entry shows only on a row that another device holds. The replica path has the same entry ((Task) 312); there it applies at once with no confirm, and `u` undoes it. |

### States

| State | What Strike does |
|---|---|
| Offline, connecting, signed out | Every write refuses at once with one line, as on the old path. |
| Frozen organization | Every write to it refuses: "Operations froze … for a migration." |
| Protocol too old | Every write refuses: "Update Strike." |
| Protocol too new | Every write refuses until Operations is updated. |
| Command sync turned off during the run | Every write to that organization refuses. Restart Strike. |
| Unit too large | Strike keeps its last complete copy and reports `unit_too_large`. |
| Unit deleted in Operations | Strike stops holding it and reports `unit_deleted`. |

A closed state also adds one entry to the failure log, so the badge shows it.

### Command Line and Inbox

- With a running Strike, `strike apply`, `finish`, `start`, `reset`, and `session finish` go to its inbox. The request id is the command id. A replay of a request after a stop applies one time.
- `strike session finish` beside a running Strike reads the rows of the files of the engine, when the recorded mode of this machine is `commands`. The old store stopped at the move, so it does not hold the rows that came later.
- With no running Strike, the command line opens the engine, waits for the snapshots of the held units, sends one command, waits for the answer, and stops. It refuses when Strike is offline or signed out, and when an organization is frozen. It never writes the old store.
- `strike target show` reads the files of the engine and makes no call. It writes nothing, also when a file does not load.

### Limits

- Undo and redo work in this process only. Undo after a restart is not available.
- The finish menu runs each step (finish, follow up, skill check, verify failed, progression) as its own gesture, so each step is its own command. A step of more than one verb (verify failed: rename, mark, finish) is one command.
- A place (the place picker) and the other writes of the Operations page have no undo.
- `R` (resubscribe) does nothing: the subscriptions are always live.
- A unit snapshot holds at most 2000 live targets. A larger unit is read only up to its last complete copy.
- The cache of a unit that the machine does not hold is empty: the Operations page reads one snapshot for each write that needs a revision.

## Source

The source of this text is `apps/strike-tui/docs/command-sync.md` in the Main repository. The contract is `packages/strike-sync/README.md`. The cutover procedure is `apps/nuu-ops/scripts/strike-sync-migrate.md`. See [[dev-knw-strk-strike]] for the model of a target.
