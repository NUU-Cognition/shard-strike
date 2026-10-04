---
description: "The Strike–Orbh tie — bindings, the launch chain, the composed prompt contract, and what a launched session may assume"
orbh-sessions:
  - "[[964ec85a-6ca8-4405-a5cb-3855486fb7c4]]"
  - "[[c174bf33-b199-4688-b89b-f26247582f78]]"
  - "[[75c5326b-39b0-41e6-8de1-81c39b3198cf]]"
  - "[[00c93799-6730-4346-b597-22b2550b2cc1]]"
  - "[[e1ce37aa-53fe-47f2-85c2-5a664ade7ae7]]"
  - "[[8f4b956a-490e-4b34-a91b-0e3faf67ec5d]]"
  - "[[c4be5d77-f1b2-4e74-bea0-a4d34dc2f60a]]"
  - "[[99bfea21-ee80-481c-ba58-e29cd81aaf3d]]"
  - "[[3fb6e594-a179-4c0b-ba67-833b2ccaea67]]"
  - "[[01286b97-7ae5-4fb6-ab72-1efcbca3c525]]"
  - "[[61b5f523-537e-425e-ab9a-002b3c2dac39]]"
---

# Knowledge: Strike Sessions

Strike launches Orbh agent sessions against single targets. This file is the contract between the two. Read it when a session prompt says it was launched from Strike, or when you work on the launch path.

## What a Launched Session Is

An ordinary interactive Orbh session in a Flint's vault. Strike is not its parent and never speaks to it again. The **composed prompt is the only channel** from Strike to the session. Everything the session knows about Strike is in that first prompt.

The reverse channel does not exist either. Strike learns the session id from the launch response and records the tie on its own side. A session cannot write back into Strike.

## The Bindings Sidecar

Two maps live beside the store, in versioned JSON with atomic writes. They are never graph state: they journal nothing and sync nowhere. Source of truth: `src/orbh/bindings.ts`.

| Map | Key | Value |
|-----|-----|-------|
| `flints` | Operations subsection id | The bound Flint (`flintId`, display `name`) |
| `sessions` | Binding key (below) | The bound session (`sessionId`, `flintId`, `flintName`, `machine`) |

The session binding key survives re-pull and re-materialisation:

| Row | Key |
|-----|-----|
| Tied to an Operations task | `task:<mountId>` |
| A subsection objective | `sub:<mountId>` |
| A project backlog anchor | `backlog:<mountId>` — the mount is the project ((Task) 225) |
| Not mounted | `local:<targetId>` |

A row that is not mounted is still a member of a subsection after (Task) 215. `local:` means "no Operations record of its own", not "outside a subsection".

## The Gestures

The whole layer sits behind the **Agent Sessions** setting. The setting defaults on. While it is off, the actions are hidden and the status feed stops; the bindings file stays on disk. The people layer is always on. Settings has no People off toggle.

| Key | Action |
|-----|--------|
| `o b` | Bind a subsection objective row to a registered Flint. The bind also installs the Strike shard into that Flint when it holds none |
| `o l` | Launch a session on the selected target |
| `s` … `a` `o` | Launch Session Together: one session over every marked target. The set must agree on one bound Flint. (Task) 230 prevents automatic Linked Tasks groups in this version. |
| `f`, or `o f` | Show the bound session. It focuses a live Obsidian tab or rewakes a sleeping session in a new focused tab. `f` is the plain tree key ((Task) 226); the fight menu moved to `F`. |
| `o R` | Focus the session and arm the return — when that session goes back to working, Strike raises its own window by itself. Press `o R` again on the same row to cancel it. The menu row shows on a waiting session only. The Focus Return setting gives `f` and `o f` the same behaviour ((Task) 231) |
| `o w` / `o W` | Reopen the bound session in a new Obsidian tab, focused / in the background. A sleeping session relaunches on the same id; a live session whose tab is gone reattaches and re-binds |
| `o r` | Resume an errored or finished session on its own profile |
| `o i` | Link an existing session by pasted id |
| `o x` | Unlink the bound session. On a row whose session is bound on another computer, the entry is **Detach Remote Agent Session**: after a confirm, every peer stops showing that session at once, and the source computer unlinks its binding when it next runs Strike ((Task) 277, (Task) 312) |
| `o X` | Release the doing claim of another computer, after a confirm (`y`). The entry shows only on a row that another computer holds. Both sync paths have it ((Task) 300, (Task) 312) |
| `I` | Open the Sessions Tray: the bottom panel grouped by session first ((Task) 303) |

The `o` menu also carries adopt (`a`), switch (`s`), copy id (`y`), close (`c`), and launch in Flint (`F`).

### The Bind Installs The Shard

A bound Flint runs every agent session for that subsection, and four of the
five launch modes tell that session to run `flint shard start strk`. A Flint
with no Strike shard therefore breaks the session's first command. `o b`
repairs that at the moment the tie is made ((Task) 257). Source of truth:
`src/orbh/flint-shard.ts`.

- **The check** is one filesystem read of `<flintPath>/Shards/Strike/init-strk.md` —
  the file `flint shard start strk` reads, so its presence IS the shard. A dev
  copy alone reads as absent, and installing from that dev copy is the repair.
  An unreadable Flint root reads as `unknown`, never as absent, by the same
  rule the registry readers follow.
- **The install** is `flint shard install strike --with-deps -p <flintPath>`,
  spawned non-blocking with process exit as the event, exactly as `openFlint`
  runs `flint open`. The source stays the bare name, so the CLI resolves it: a
  dev copy inside that Flint installs offline, and a Flint with no copy
  expands to `NUU-Cognition/shard-strike`. `--with-deps` carries the three
  declared dependencies, of which `NUU-Cognition/shard-projects` is the one
  that matters here — `wkfl-strk-strike_start` delegates to
  `wkfl-proj-scope_task`.
- **The order.** The sidecar write and the `Bound "<row>" → <Flint>` flash come
  first. The binding is a local fact, and the shard check is a repair that
  follows it; no failure of the check can fail a bind. The operator then sees
  `Installing the Strike shard into <Flint>…` and the outcome. A `present` or
  `unknown` answer says nothing more.
- **One install per Flint directory.** The guard is keyed by the resolved
  path, so binding several subsections to one Flint in a row never runs two
  CLI calls over the same directory.

### A live session with no tab

An interactive session is a detachable daemon; the Obsidian tab holds only an attach client. When Obsidian closes, the tab dies and the agent keeps running. The session stays live (working or attention) and its Obsidian binding — `metadata.obsidian` on the session, keyed by the pane's pty pid — goes stale. `o f` then fails with "tab is gone" and points at `o w`. `o w` posts the resume route; the new pane types `flint orbh i <target> -c <id>`, which attaches the live manager and re-persists the binding, so `o f` and `flint orbh focus` work again. No second session is created.

A bound row carries a live three-state glyph: attention, working, error.

### The Sessions Tray

`I` opens the **Sessions Tray** ((Task) 303). It is the third mode of the
bottom panel that the doing peek (`D`) and the skill check tray (`S`) share.
It keeps the doing peek's division — one headed block per open scoped tab,
named as the tab bar names it — and inside each block every session is a
header with its rows under it. A session header steps in under its tab by the
doing peek's plain indent. Only the rows wear the tree view's guides (`├─`,
`└─`) under their session. Source of truth: `buildSessionTrayRows` in
`src/tui/tree-model.ts`; the renderer is the doing peek's
(`src/tui/views/doing-peek.ts`, `sessionTray`).

```
1: Strike
  <mark> Tray work · claude/opus · NUU Operations
  ├─ <state> Write tray
  └─ <state> Plan tray
  <mark> Sync · claude/opus · NUU Operations
  └─ <state> Fix sync
```

- **Which sessions.** A session shows when at least one live row wears it as
  its SELECTED session in the bindings sidecar. Every session gesture on a row
  targets the selected session, so a row under a header always acts on that
  header's session. A session that is only a non-selected link has no row to
  act through, and the tray leaves it out. A `done` session is left out: the
  sidecar keeps the bindings of closed sessions. The list is store-wide.
- **Which block.** A session sits in a tab's block when one of its rows does,
  by the doing peek's rule (`doingGroupKeysFor`). A session with rows under two
  tabs heads one block in each, with that block's rows. A row that no open tab
  claims renders nowhere, as in the doing peek. With no tab open the tray is
  flat: the sessions are the top level.
- **Order.** Sessions sort waiting, working, error, sleeping, then by title.
  Rows inside a session sort doing, open todo, blocked todo, done.
- **The session header** reads `[elapsed] <session mark> [lifecycle] Title · runtime/profile · Flint`.
  A member row leads with its own state glyph, because the header already
  carries the session mark — the Add To Session picker (`o g`) paints its rows
  the same way.
- **Keys.** `j`/`k` select rows and headers, `h`/`l` fold a tab block or a
  session (for this run only, apart from the doing peek's folds), `Z` folds
  every session, `H`/`L` jump header to header. `enter` on a tab header goes
  to the tab. `enter` on a session header and `f` focus the session; `enter`
  on a row goes to the task. `o` opens the agent menu. `space`, `ctrl-space`, `shift-space`, `a`,
  `e`, `u`, `ctrl-r`, `+`/`-`, and `F` act as in the doing peek. `g` starts the
  number jump (below). `I`, `esc`,
  or `q` closes; `D` and `S` switch the mode. A header acts through its first
  member for `f`, `o`, and `enter` only; a status gesture on a header does
  nothing.
- **Search.** `/` opens the doing peek's search over the tray: the same
  modal, the same status line, and the same keys. A session whose title holds
  the query keeps all its rows; any other session keeps only the rows whose
  title holds it. The tree is built again over the matches, so the tab and
  session headers stand as context and the guides stay true. The cursor
  (`tab`/`shift-tab`, `↑`/`↓`) stops on the matches only, `enter` puts the
  panel's cursor on the match, and `esc` cancels. The search reads the folds:
  a row under a folded header is out of the search. Source:
  `sessionTraySearchMatches` in `src/tui/tree-model.ts`.
- **Number jump ((Task) 306).** `g` puts a number at the start of each
  session header, after its indent: 1, 2, 3, … from top to bottom. The
  numbers come from the rows the tray shows, so a session under a folded tab
  block has no number, and a session that heads a block under two tabs has
  one number for each header. Type the digits of a number, then press
  `enter`: the cursor goes to the session header with exactly that number.
  `1` is session 1 and `12` is session 12; there is no prefix match and no
  jump before `enter`. A number with no session flashes `No session <N>` and
  keeps the jump on. `backspace` erases one digit, and a leading `0` is not
  typed. `esc` or `g` ends the jump with no move. Any other key ends the jump
  and then acts as usual, and a click on a row ends it too. While the jump is
  on, the exact match wears a solid accent block, a number the digits can
  still reach is in the accent, any other number is dim, and the hint line
  reads `Go to session: <digits> · enter Go · bksp Erase · esc Cancel`. The
  jump ends when the tray closes, when the panel mode changes, and when a
  modal opens. Source: `sessionTrayJumpIndexes` and `sessionTrayJumpTarget`
  in `src/tui/tree-model.ts`.
- **Off setting.** With Agent Sessions off, `I` flashes the setting and opens
  nothing.

## The Launch Modal

`o l` opens one panel with three stages. Esc steps back one stage and closes from the first.

| Stage | Content |
|-------|---------|
| 1 Profile | The configured profile list, or the whole machine catalog while that list is empty. Digits `1-9` select a visible row while the filter is empty; typing filters. |
| 2 Account | The runtime's accounts, effective default first and preselected. Skipped when the runtime has at most one account. |
| 3 Prompt | A composer for the operator's extra text. `Tab` cycles the launch mode (Strike Start, Discuss First, Create and Do Task, Notepad, None); `shift-tab` cycles back. The hint shows the current mode. Enter launches; shift-enter launches and focuses. |

## The Launch Chain

Launch travels over HTTP, not over a process spawn.

```
Strike
  → POST /workspace/orbh/launch   on the bound Flint's flint-server
  → (fallback) POST /launch       on the machine helper broker, port 10003
  → NUU Flint plugin control server
  → flint orbh i --account <name> ...   typed into a pty
```

The wire carries `{ profile, prompt, account, focus, at }`. The response returns the pre-allocated session id. An ok response with no session id means the Obsidian window runs an outdated in-memory plugin.

Focus is different: it shells out to `flint orbh focus <id> --at <scope>`.

### A closed Flint

Every route above ends in the NUU plugin inside the bound Flint's Obsidian window. When that window is closed, Strike opens the Flint before it launches, resumes, or focuses ((Task) 227). The source is `src/orbh/open-flint.ts` and the `withOpenFlint` wrapper in `src/tui/app.ts`.

- **Detection.** Strike reads the Blacksmith resources file (`~/.nuucognition/blacksmith/resources.json`) for a live `obsidian-manager` row whose `flintPath` is the bound Flint. No row means closed, and Strike opens first. An unreadable file means unknown, and Strike keeps the normal route and reacts to the failure text instead: the flint-server's `bridge_unavailable` text ("Obsidian operations are only available when an Obsidian manager is registered") or the helper broker's "knows no live vault".
- **The open.** Strike runs `flint open <name>`. The Flint CLI owns the policy: the hot path asks a live Obsidian manager to `open-vault`, the cold path starts Obsidian in the machine's Blacksmith Flint and brokers the target open through it. The plugin's `open-vault` is a same-window switch, so the hot path turns another Flint's window into the target; Strike inherits that rule from the CLI.
- **The wait.** The CLI exit is not the signal — the new window's manager registers seconds after the switch. Strike watches the resources directory with `fs.watch` until the manager row appears, with one 45 s bound and no polling, then runs the gesture once more.
- **The gestures.** `o l` and Launch Session Together, the resume core behind `o w`, `o W`, and `o r`, and `o f` (a closed Flint skips both focus routes and goes straight to the reopen, which opens first). An open Flint takes the old path with no extra spawn.

## The Composed Prompt Contract

Source of truth: `src/orbh/preamble.ts`. The prompt has four parts, joined by single spaces, always on **one line** — the plugin types it into a pty as one shell-escaped argument. The third part is present only for a tied launch.

### 1. The preamble

The preamble names the task, then the **ancestry** — the path of parent rows from the subsection down to the task. The common shape, where the subsection objective stands above the row:

```
This session was launched from Strike for task '<title>' (id: <targetId>). Its ancestry: '<subsectionTitle>' (subsection id: <subsectionId>) > '<ancestor>' > … > '<title>'.
```

A row whose subsection stands nowhere above it — the subsection came from a scope or a `pushBinding` — keeps both clauses:

```
This session was launched from Strike for task '<title>' (id: <targetId>) under subsection '<subsectionTitle>' (id: <subsectionId>). Its ancestry: '<ancestor>' > '<title>'.
```

A row with no ancestors:

```
This session was launched from Strike for task '<title>' (id: <targetId>) under subsection '<subsectionTitle>' (id: <subsectionId>).
```

A row with neither. (Task) 215 made every row a member of a subsection, so a new prompt of this shape is not expected. Older prompts may still carry it.

```
This session was launched from Strike for task '<title>' (id: <targetId>).
```

| Field | Source | Notes |
|-------|--------|-------|
| `<title>` | The Strike target's title | Operator text; may contain any characters except newlines |
| `<targetId>` | The Strike target `_id` | Always present |
| `<subsectionTitle>` | The subsection objective row's title | Present only with the id |
| `<subsectionId>` | The Operations subsection id | Present only with the title |
| `<ancestor>` | A parent row's title | Title only — see below |
| `<opsTaskId>` | The `mountId` of a row tied to an Operations task | Present only for a tied row — see "The Operations clause" |
| `<opsOrg>` | The Operations org of that task | Present only with the task id, when the org is known |

The subsection clause is all-or-nothing. A subsection named without its id is not addressable, so both drop together. When the objective heads the ancestry, it carries the subsection id there and the `under subsection` clause drops.

The subsection is resolved in this order: the row's own `ops-subsection` mount, then a scope grounded in a subsection, then `pushBinding.subsectionId`, then the inherited push destination.

### The Operations clause

A row tied to an Operations task (`mountKind: "ops-task"`, a mount that resolves) adds one clause inside the parentheses of its task phrase ((Task) 309). Source of truth: `opsTaskClauseFields` and `opsTaskClause` in `src/orbh/preamble.ts`.

```
task '<title>' (id: <targetId>; Operations task id: <opsTaskId>, org: <opsOrg>)
task '<title>' (id: <targetId>; Operations task id: <opsTaskId>, org: <opsOrg>, data: dev)
task '<title>' (id: <targetId>; Operations task id: <opsTaskId>)
```

- The org is the tie's `mountOrg`, normalised by `normalizeOrgSlug`: an absent mount org means the home org. When Strike does not know the home org yet (the Operations data has not loaded), an absent mount org drops the org part. The id still resolves, because `ops` finds the org of a task from its id.
- `data: dev` shows only when Strike runs on dev data. Then the id is a dev Operations id. Prod data adds nothing.
- An `ops-subsection` mount, an `ops-backlog` mount, a dangling mount (`mountMissing`), and an untied row add no clause. Their prompts are byte for byte the same as before (Task) 309.
- A together launch builds each task phrase alone, so each tied target carries its own clause.

The session gets the full context of the Operations task with one command. The Operations sentence (part 3) names it in every launch mode:

```bash
ops task context <opsTaskId> --org <opsOrg>
```

It prints one Markdown block: the facts of the task (status, priority, place, owner, creator, URL), the project description, the task description, and every comment of the Updates timeline, oldest first, with no Waybill event. The command wraps the gateway op `agent.taskContext`. `--json` prints the op result. See the Operations shard (`knw-ops-cli`) for the command reference.

**Ancestry rules.**

- The path runs top-down: the subsection first, the direct parent last, the task itself as the tail.
- Ancestors carry **titles only**. Strike has no reverse channel, so an id for a row the session can never address would be prompt length spent on nothing. Only the task and the subsection carry ids.
- The climb ends at the subsection objective when that row stands above the target. Otherwise it ends at the nearest grounded ancestor. ((Task) 215 removed the flag mount, so no flag row stands in the path.)
- The path is never elided. A deep row sends its whole chain.
- The whole preamble stays on one line, as the rest of the prompt does.

**More than one target.** Select mode marks a set and the bulk menu launches one session over all of it (`s`, then `a`, then `o` — Launch Session Together). The preamble then says the count, says the targets were launched together, and names every one of them with its own id:

```
This session was launched from Strike for 2 tasks launched together: task '<titleA>' (id: <targetIdA>) and task '<titleB>' (id: <targetIdB>). Their shared ancestry: '<subsectionTitle>' (subsection id: <subsectionId>) > '<ancestor>'.
```

- The list runs in row order. The topmost marked row leads.
- A subsection clause every target shares is said once, before the list. Otherwise each target carries its own.
- One shared ancestry clause covers the set when every target has the same path. The tail is the direct parent, not a task. When the paths differ, each target gets `Ancestry of '<title>': …` instead, ending in its own title.
- The Create and Do Task and Notepad instructions take their plural form ("these tasks"). Strike Start is unchanged.
- One marked row composes exactly the single-target prompt.

Strike does not tell the session what to do with a set. The prompt states the tasks and their ids; the session decides how to handle them.

### 2. The launch mode instruction

The operator picks a launch mode with `Tab` in the prompt stage. The default is Strike Start. Each mode appends one sentence, or none.

| Mode | Instruction |
|------|-------------|
| Strike Start (default) | ``Run `flint shard start strk`, then follow the wkfl-strk-strike_start workflow before other work.`` |
| Discuss First | ``Run `flint shard start strk`, then follow the wkfl-strk-discuss_start workflow before other work.`` |
| Create and Do Task | ``Run `flint shard start strk`, then follow the wkfl-proj-create_and_do_task workflow for this task before other work; record the Strike target id and the subsection id in the task's Notes.`` |
| Notepad | ``Run `flint shard start strk`, then follow the wkfl-ntpd-start workflow with this task as the topic before other work.`` |
| None | No instruction. The preamble goes alone, then the Operations sentence of a tied launch, then the extra text. |

Every mode except None loads the Strike shard first, so the session can read this contract and record the ids it was given. A tied launch in the None mode loads it through the Operations sentence (part 3).

**Discuss First is a wrapper, and the prompt does not say so.** The instruction names `wkfl-strk-discuss_start` and nothing else. That workflow discusses the target in the chat, writes no Mesh artifact while it talks, and hands off to [[dev-wkfl-strk-strike_start]] only when the operator agrees. The handoff is the workflow's decision, not the prompt's, so the prompt stays one sentence like every other mode. `Tab` puts the mode second in the cycle, right after the mode it wraps.

**The plural form.** Discuss First takes a plural sentence for a together-launch: ``…then follow the wkfl-strk-discuss_start workflow for these tasks before other work.`` Strike Start is still the one task-facing mode with no plural, because it names a workflow and nothing else.

**Headless launch ((Task) 281).** `alt-h` in the prompt stage toggles a headless launch. A headless launch offers exactly three modes — Strike Start, Discuss First, Create and Do Task — and each instruction swaps to its headless form: `flint shard hstart strk`, then `hwkfl-strk-strike_start`, `hwkfl-strk-discuss_start`, or `hwkfl-strk-create_and_do`. A notepad is a conversation surface and `none` leaves no return contract, so neither exists headless. The launch shells out to `flint orbh launch <profile> <prompt>` in the bound Flint's root — no Obsidian window anywhere on the path — and Strike parses the session id from the CLI's `ID:` line.

### 3. The Operations sentence

A launch that carries a row tied to an Operations task adds one sentence after the mode instruction ((Task) 309 review, (Task) 341). It stands in **every** mode, None included: the mode chooses a workflow, and the context read and the updates belong to no one mode workflow. Source of truth: `opsContextInstruction` in `src/orbh/preamble.ts`.

```
The task is tied to an Operations task: run `ops task context <opsTaskId> --org <opsOrg>` before other work, and follow the wkfl-strk-ops_do workflow of the Strike shard (`flint shard start strk`) to start it and post its updates.
```

- **Several tied rows.** A together launch names the command of each tied task in row order: ``2 tasks are tied to Operations tasks: run `ops task context a --org o` and `ops task context b --org o` before other work, and follow … to start them and post their updates.`` One tied row among several reads `One task is tied to an Operations task: …`.
- **No org.** A context with no org drops `--org`, as the clause does.
- **Headless.** The sentence names `hwkfl-strk-ops_do` and `flint shard hstart strk`.
- **Dev data.** The sentence adds ``The id is a dev id: run `ops mode` first, and when it does not show dev data, start each command with `ndv run`.`` (``The ids are dev ids`` for several).
- **Untied.** A launch with no tied row adds nothing. Its prompt is byte for byte the prompt before (Task) 309.

The workflow [[dev-wkfl-strk-ops_do]] (headless [[dev-hwkfl-strk-ops_do]]) runs beside the workflow of the mode. It finds the linked task (the clause, the `ops-task` field of a Mesh task, the operator, or the Flint place of a deliverable), reads its context, starts it when the implementation starts, and posts one place-relative update at the start, at a blocker, at a review stop, and at done. It writes only to the linked task and ends each comment with the agent signature line.

### 4. The operator's extra text

Whatever was typed in the prompt stage. Newlines collapse to spaces. It comes last, so it is the operator's final word and may override the instruction above it.

## What a Launched Session May Assume

**May assume:**

- The task title and target id are accurate at launch time.
- The subsection title and id, when present, name a live Operations subsection.
- The Flint it woke in is the one bound to that subsection.
- The ancestry, when present, names live Strike rows that stood above the target at launch time.
- The operator was looking at that exact row when they pressed `o l`.
- When the task phrase carries an Operations clause, the row was tied to that Operations task at launch time, and the id is an Operations task id of the named org (a dev id when the clause says `data: dev`). The prompt then also carries the Operations sentence.
- That a prompt naming more than one task is deliberate. The operator marked that set and launched it as one piece of work, and one session holds all of it.

**May not assume:**

- That the ids still resolve. The operator can delete or re-pull a row after launching.
- That the tasks of a together-launch are linked in Strike. (Task) 230 makes them share only the session in this version.
- That an Operations task exists when the task phrase has no Operations clause. Such a row may hold no mount at all, or a mount of another kind.
- That the Operations task still holds the title it had at launch. Read it with `ops task context`.
- That a subsection clause is present. Every row created after (Task) 215 has a subsection, so the clause is expected; an older prompt, or a row whose subsection no longer resolves, may still lack it. Tolerate the missing clause instead of failing.
- That an ancestry clause is present. A row that hangs under nothing has none.
- That the ancestry names every parent. The path is one line up the graph, and a target may be required by several rows.
- That the extra text relates to the named task at all.

Treat every id as an opaque string. Quote it, pass it through, and never construct or guess one.

## The Lifecycle Label

The one channel a session has back to a Strike row is the Orbh session interface. Strike reads one key from it, `strike-lifecycle`, and renders it as a label glyph right after the session status glyph. Source of truth on the TUI side: `src/model/glyphs.ts` (`GLYPHS.agentLifecycle`, `lifecycleLabel`) and `src/orbh/status.ts`.

| Value | Glyph | Codepoint |
|-------|-------|-----------|
| `scoped` | nf-md-telescope | `U+F0B4E` |
| `pre-scope` | nf-md-text_to_speech | `U+F050A` |
| `discussing` | nf-md-message_processing | `U+F0366` |
| `review` | nf-md-notebook | `U+F082E` |
| `input`, or any other non-empty value | the operator-chosen `other` mark | `U+F1738` |

**`pre-scope` and `discussing` are two states.** `pre-scope` (󰔊) means no task exists yet and the session is still deciding what the task should be. Only [[dev-wkfl-strk-discuss_start]] writes it. `discussing` means a task exists and its proposal is under discussion. The wire value carries the dash; the TUI glyph key is `preScope`, because the key set is a TypeScript identifier list. A value written as `prescope` or `pre_scope` falls through to the `other` mark.

**Transport.** The value rides the same feed as the status glyph. Over SSE it is `session.interface["strike-lifecycle"]` on the `snapshot` and `session.changed` payloads; a write produces `changedFacts: ["interface", ...]`, which the feed treats as relevant. Over the filesystem fallback it is `ext.orbh.slices["core:interface"]["strike-lifecycle"]` in the spool snapshot. Strike keeps the raw string on the session status entry and maps it at render time.

**Display rule.** The label shows the last set value while the session state is `attention`, `sleeping`, `done`, or `error`. It hides while the state is `working`, and it hides on a row whose target is effectively done (the frozen mark). A closed session keeps its last label. The empty string means no label.

**Who writes it.** The session, by the rule in the init: set the label when a piece of work is finished and the session is about to wait, never during. Strike never writes it, and the close skill does not clear it.

## The Focus Return

Focus is one-way by default: Strike shows the session, and the operator walks back by hand. `o R` closes the loop ((Task) 231). It focuses the session and arms a one-shot watch on it; the moment that session goes `attention → working`, Strike raises its own window. The setting **Focus Return** gives plain `f` and `o f` the same behaviour, and the chord arms it either way.

The agent menu is built from its own state-aware item list, not from the keybind table, so the row **Focus and Return** appears there only while the session waits — a session that already works has no `attention → working` edge left, exactly as park shows only on a live session. The chord itself is always registered, so the palette and the help list it in every state.

The trigger is the same observed transition the fight hit counter reads, so nothing new polls. Only one arm exists at a time, and four things spend it: the transition firing, a second focus gesture replacing it, a second `o R` on the same row, and a 15 minute expiry. An explicit `o R` on a session that is already working refuses, because there is no edge left to wait for.

An earlier rule spent the arm on ANY key pressed in Strike, on the reading that a key means the operator came back. That rule was wrong and it broke the feature: the operator presses keys in Strike after they arm and before they leave, and every one of those keys killed the arm. Strike also cannot see when its own window loses focus — the terminal reports no focus events — so "the operator came back" is not observable. The explicit second `o R` replaces it. The rewake paths never arm: a rewake reaches `working` within seconds and would snatch focus off the tab it just opened.

**Raising Strike is not a terminal operation.** Strike draws inside a terminal emulator and owns no window, so the raise leaves the terminal protocol entirely. `foot` implements only the report, push and pop window sequences and has no raise, and Wayland gives a client no self-raise. Source of truth: `src/orbh/raise-self.ts`. It resolves Strike's host window once, by walking the process parent chain to the first process the platform lists as a window owner, and then picks the first backend that works:

| Backend | When | What it runs |
|---------|------|--------------|
| `command` | The `focusReturnCommand` setting is set | That command through `sh -c`, with `{pid}` and `{address}` substituted |
| `hyprland` | `HYPRLAND_INSTANCE_SIGNATURE` is set | `hyprctl eval 'hl.dispatch(hl.dsp.focus({ window = "address:0x…" }))'`, then a cursor warp into the window |
| `darwin` | `process.platform` is `darwin` | `open -a <app>`, with the application read from `TERM_PROGRAM` |

**`hl.dsp.*` only BUILDS a dispatcher. It must be wrapped in `hl.dispatch(...)` to run.** A bare `hl.dsp.focus({...})` answers `ok` and has no effect whatever. This cost three rounds of debugging: the log recorded three commands, all exit code 0, and the window never moved. Proven on the machine — `hl.dsp.cursor.move` left the pointer where it was, and the same call inside `hl.dispatch` moved it.

One focus dispatch is the whole job on Hyprland: a window on a hidden special workspace has that workspace shown AND the window focused by that single call. An earlier version emitted `toggle_special` first; that step was unnecessary, and its `name` argument was ignored, so it opened the unnamed `special:special` workspace instead of the operator's scratchpad. The second step is a cursor warp, because `input:follow_mouse` otherwise takes the focus straight back.

The operator's command outranks every built-in backend on purpose. The exact call is version-specific: Hyprland 0.56 moved `hyprctl dispatch` onto a Lua API and rejects the classic `hyprctl dispatch focuswindow address:0x…` string form, so the operator must be able to fix this without a Strike release. A machine with no backend and no command refuses with a flash that names the setting; it never fails silently.

## Replication

A held subsection replicates to every org peer that holds it ((Task) 224), so a target a session works on can change under it. What that means for a session:

- **What travels** is the local truth of every row in the unit — title, body, status, `requires`, memberships, dates, priority, rarity, kind, modifiers, overlay, marks, labels, `tieBreak`, archive and completion moments, the mount identity, and `pushBinding`. Derived fields (`readiness`, `tiePending`, `ownerUserId`, `mountMissing`) never travel; each client re-derives them.
- **The bindings sidecar does NOT travel.** `strike-orbh.json` is machine-local. A peer sees the row and a display-only remote session state ((Task) 216): the state glyph, the lifecycle label, and the machine name. The state gives the peer no control of the session.
- **Remote detach ((Task) 277, (Task) 312).** A peer can detach the remote session from a row with `o x`. The request goes to the session state of the source computer on the server, in every unit of the org where the state has the same binding token, also in a unit that the peer does not hold. Every peer stops showing a state that has a pending request, so the detach takes effect at once, also when the source computer is off. The source reads the request when it next runs Strike, unlinks its binding when the binding token still matches, and acknowledges. A source that reports a new binding token reports a new fact, and peers show it again. The server keeps a state for seven days after its last write, and the request counts as a write. When the source returns after more than seven days, it reports its binding again and the remote session comes back on peers.
- **Foreign doing claims ((Task) 243, (Task) 312).** A claim of another computer locks start, finish, and launch on the row. `o X` releases it from any computer. On the replica path the server mutation `strikeReplicas:forceReleaseDoingClaim` deletes the claim of that computer on the row in every unit of the org; the other computer keeps its own `doing` and claims the row again only from a new start gesture. On the command path the command is `ForceReleaseDoing`.
- **The claim replay rules ((Task) 312).** These rules keep a released claim released when the source computer restarts and sends an old claim again.
  - Each start gesture has one random intent id. The store keeps it with the claim intent of every unit of the row, and the claim carries it to the server. A new start has a new id.
  - A release keeps the ids of the released claims for 14 days in `strike_doing_claim_releases` (one record for each org, row, and source computer). The server refuses a claim whose id is in the record (`refused`), and the intent is spent.
  - Strike retires a claim intent that is older than 7 days without a send, and the sync panel names the drop. The server keeps an id longer than any client can send it, so a replay after the 14 days cannot occur.
  - A store from before the ids gives each old intent a new id at load and saves the store at once, so every later load reads the same id.
  - A claim with no id comes from an older Strike. It works while no release record exists for the row and that computer. When a record exists, the server refuses it with `strike_claim_upgrade_required`, and the operator updates Strike.
  - Against a deployment without the ids, Strike sends claims without them for 10 minutes, then tries the ids again. A `strike_claim_upgrade_required` refusal makes it send the ids at once.
- **The `sub:` rule.** A unit's objective row travels under the derived identity `sub:<subsectionId>`, never a minted UUID, and an arriving `sub:` row maps onto the local anchor for that subsection. A launch on an objective is therefore a launch on this machine's anchor row.
- **Last writer wins**, per row, by server-stamped `rev`. A peer's newer edit replaces the local row; a local edit whose push is still owed keeps the local version. So a session that has held a row's title in its prompt for a while may be looking at a stale title.
- **Tombstones.** A peer's delete arrives as a tombstone and deletes the local row, sweeping edges and repairing overlays. A session bound to a row a peer deleted loses its target; the lifecycle label then reads as a dangling binding, and the binding is cleaned up on the next sweep.
- **The sink.** Replication is best-effort behind the local commit, and every failure reaches the sync failure sink under phase `replica`. A refused read leaves the unit local; it never empties.

## Headless Sessions and the Return-Result Chat

A headless session has no pane and no Obsidian binding ((Task) 281). Its conversation with the operator is the **return-result chat**: Strike derives it from the session's spool and renders it as a modal chat surface.

**What the chat shows, in time order:**

- The launch prompt (`ext.orbh.prompt`), as the operator's first message
- Every returned result (`ext.orbh.runs[].result`), as the session's replies
- Every inbox message (`ext.orbh.slices["core:inbox"]`): `from: "human"` renders as the operator, a `Strike: ` prefixed text renders as a system line, and a session-id sender renders as a peer line

**The gestures.** `o t` opens the chat on any bound row. On a headless session, `f`, `o f`, `o w`, and `o W` open the chat too — there is no tab to focus or reopen. Enter in the chat composer sends the text with `flint orbh message send <id> <text> --revive`, spawned in the bound Flint's root. The message is durable in the spool, an awaiting session wakes on it through the machine orchestrator, an ended one revives, and a working one reads it at its next turn boundary. The next returned result appends to the chat through the same status feed events every glyph rides.

**What a headless session must do:** end every turn with `return --await` (or `--finish`), write results as chat replies, set the lifecycle label before every await, and never block on `session ask`. The contract is in `hinit-strk.md` and the three `hwkfl-strk-*` workflows.

## Failure Modes

| Symptom | Cause |
|---------|-------|
| The prompt names no subsection | The row has no resolvable subsection context — it predates (Task) 215, or its subsection no longer resolves |
| The session runs under the wrong account | An outdated flint-server or in-memory plugin dropped the unknown `account` field silently |
| Launch reports no session id | The Obsidian plugin is outdated; reload Obsidian |
| Launch cannot reach anything | No flint-server for that Flint and the helper broker is down — `flint system helper router start` |
| "Opening <Flint>…" then "Opened <Flint> but its Obsidian manager did not register in 45s" | The Flint's Obsidian window is closed and `flint open` did not bring a manager up in time — a slow cold start, or the NUU plugin is disabled in that vault. Open the Flint by hand and retry |
| "Opening <Flint>…" then "Could not run flint" | The `flint` CLI is not on Strike's PATH |
| Focus fails with "Obsidian tab is gone" on a live row | The tab died under a detachable session (Obsidian closed). `o w` reopens and re-binds it |
| A reopened tab shows only a shell prompt | The plugin typed the resume command before the shell was ready. Close that tab and press `o w` again |
