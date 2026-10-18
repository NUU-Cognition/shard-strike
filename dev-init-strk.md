---
required-reading:
  - "[[dev-knw-strk-strike]]"
  - "[[dev-knw-strk-sessions]]"
orbh-sessions:
  - "[[c174bf33-b199-4688-b89b-f26247582f78]]"
  - "[[acefe761-97a3-4421-86ee-08717b7cb565]]"
  - "[[3fb6e594-a179-4c0b-ba67-833b2ccaea67]]"
  - "[[61b5f523-537e-425e-ab9a-002b3c2dac39]]"
  - "[[d98d16ae-1388-4228-8610-6954998a395a]]"
---

# Strike

Strike is a local-first terminal app for a personal **target graph**. It ties selected parts of that graph to NUU Operations, and it launches Orbh agent sessions against single rows. This shard is the context a launched session needs to understand where it came from and what the operator was looking at.

Load this shard when a session prompt says it was launched from Strike, or when the work touches Strike itself.

## The Two Concepts

| Concept | Meaning |
|---------|---------|
| **Subsection** | One Operations subsection, held locally. A Strike subsection is a 1:1 match to an Operations subsection. There is no local-only subsection, and no target outside a subsection. |
| **Root** | The subsection that orients the current tab. A tab holds one or more subsections and one of them is the root; `]` rotates the root through the tab's set. |

The subsection is the unit ((Task) 215). The operator starts Strike by choosing a subsection.

Operations enters the graph through one door: the **subsection picker**. It is a flat, searchable list of `Project / Deliverable / Subsection` rows over every organisation the credential belongs to. Every gesture that needs a subsection opens it (`n`, `W s`, `W r`, `w s`, `a L s`, `:pull --subsection`, `:tab new`). `n` opens a NEW tab rooted at the choice; `W r` roots the CURRENT tab at it. With no tab open, Strike shows "Choose a subsection to begin" and opens the picker once; signed out it shows a sign-in hint instead. There is no project browser in Strike; the earlier Catalog was removed in (Task) 194. The **Operations page** (`v O`) manages projects, deliverables, subsections, and tasks in the terminal; see [[knw-strk-strike]].

## The Target

A **target** is the single node type. It carries a title, a status, and a requires-list that forms a directed acyclic graph. Everything else in Strike is a view over targets.

```
todo → doing → done
```

Readiness is derived, not stored: `blocked` (an open requirement holds it), `open` (workable), `closed` (done or archived).

Four mutually exclusive **kinds** refine a target: `combo`, `attempt`, `ranged`, `stage`. Two **modifiers** compose with any kind: `full` and `sideStrike`. See [[dev-knw-strk-strike]] for what each one does.

## The Mount Boundary

A target is local until it is **mounted** to an Operations record. The mount is the only place canon enters the model.

| `mountKind` | Ties the target to |
|-------------|--------------------|
| `ops-task` | An Operations task |
| `ops-subsection` | An Operations subsection — this row is the objective |

Creation is local-first. Canon is untouched until an explicit push.

## Vocabulary

A gamification setting respells static UI strings. The operator may speak either dialect. Both name the same thing.

| Internal | Gamification on | Gamification off |
|----------|-----------------|------------------|
| target | target | task |
| combo | combo | folder |
| subsection | boss | subsection |
| aim | aim | goal |
| Finishing Blow | Finishing Blow | Close Folder |

Never respell an operator's own title text. A target titled "subsection sweep" is read verbatim.

## Sessions Launched From Strike

Strike binds an Operations subsection to a Flint, then launches an Orbh session against one target inside it. The composed first prompt names the task with its id, then the ancestry — the path of parent rows from the subsection down to the task. By default it instructs the session to run [[dev-wkfl-strk-strike_start]] before other work. The operator can pick another launch mode in the prompt stage: discuss the target first, create and do the task, start a notepad, or send no instruction.

The **Discuss First** mode names [[dev-wkfl-strk-discuss_start]]. That workflow talks about the target in the chat, writes no Mesh artifact while it talks, and then wraps [[dev-wkfl-strk-strike_start]]. The scoped task carries a detailed summary of the discussion. Use it when the row is fuzzy and a task written before the talk would state the wrong problem.

Read [[dev-knw-strk-sessions]] for the binding model, the launch chain, and the exact preamble contract.

**The Operations task ((Task) 309).** When the task phrase of the preamble carries `Operations task id: <id>`, the Strike row is tied to that Operations task. Before other work, in every launch mode, read the full context of the task:

```bash
ops task context <id> --org <org>     # omit --org when the clause names no org
```

The command prints the facts of the task, the project description, the task description, and every comment of the Updates timeline, with no Waybill event. Use the description and the comments as context for the work, and carry a short summary of them into any task you scope. When the clause says `data: dev`, the id is a dev id. Start every `ops` command of that task with `OPS_DATA_MODE=dev ndv run`, for example `OPS_DATA_MODE=dev ndv run ops task context <id> --org <org>`. First run `OPS_DATA_MODE=dev ndv run ops mode --json`, and read or write nothing when its `dataMode` is not `dev`: `ndv run` alone selects only the dev launcher, and that launcher still follows an inherited `OPS_DATA_MODE` or the stored data mode ((Task) 341 review). When no command can read the task (no credential, a deleted task, a wrong data mode), tell the operator in one sentence and continue without the context. The clause contract is in [[dev-knw-strk-sessions]].

**Work with Operations ((Task) 341).** A tied launch ends its instructions with one Operations sentence, in every launch mode, None included. The sentence names the `ops task context` command of each tied task and the workflow [[dev-wkfl-strk-ops_do]]. Follow that workflow beside the workflow of the launch mode. It starts the Operations task when the implementation starts, and it posts one place-relative update at the start, at a blocker, at a review stop, and at done, through `ops comment add`. It writes only to the linked task, and it ends each comment with the agent signature line, which names the fleet and the Orbh session. It also finds the Operations work of this Flint by place: the deliverables whose Flint place (`flintAddress`, or `flintId`) is this Flint. A Mesh task that is tied to an Operations task carries the frontmatter fields `ops-task` and `ops-org`.

**Headless launches ((Task) 281).** `alt-h` in the launch prompt stage makes the launch headless: no Obsidian window, and the session's whole operator channel is the return-result chat (`o t` on the bound row). A headless launch offers three modes only — Strike Start, Discuss First, Create and Do Task — and points the session at the Strike shard and a Strike workflow. The text depends on the layout of the Strike shard of the bound Flint (NUU Orbh Task 748). For one workflow for each mode, the prompt names `flint shard start strk` and a `wkfl-strk-*` workflow, with the line "Follow the `> [!headless]` callouts of each stage." For the old layout, or when Strike does not know the shard folder, it names `flint shard hstart strk` and a `hwkfl-strk-*` workflow (see `# Headless` § Old Launch Prompts). A headless session ends every turn with `return --await` or `return --finish`; the returned result is the message the operator reads.

## The Lifecycle Label

A bound Strike row shows a session status glyph. When the session waits, that glyph says only that it waits. The **lifecycle label** says why. It is one Orbh interface key that this session writes and that the Strike TUI renders as a second glyph beside the status glyph.

```bash
flint orbh session set strike-lifecycle <value>
```

| Value | Glyph | Meaning |
|-------|-------|---------|
| `scoped` | telescope | The task is scoped and waits for the operator to say go |
| `pre-scope` | text to speech | The session discusses the target before any task exists ([[dev-wkfl-strk-discuss_start]]) |
| `discussing` | message | The session is in a conversation with the operator: a proposal review, a notepad, a design talk |
| `review` | notebook | The work is finished and waits for the operator to review it |
| `input` | other mark | The session is blocked, needs a decision, or is in any other state |

**Set the label when you finish a piece of work and are about to wait for the operator. Never set it during the work.** The TUI hides the label while the session works and shows the last value in every rest state, including a closed session. So the label you set as your last action before you wait is the one the operator sees. Do not clear it on close.

The mirror table below maps the state you leave the work in to the label. It holds for the whole session, also inside workflows owned by other shards (do_task, review_task, notepad start and continue).

| You stop and wait because | Set |
|---------------------------|-----|
| You wait during a pre-scope discussion, before any task exists | `pre-scope` |
| The task is `todo` after scoping, including the first proposal wait | `scoped` |
| You continue a proposal conversation, reply in a notepad, or discuss a design | `discussing` |
| The task is `review`, `reviewing`, or `reviewed` | `review` |
| The task is `blocked`, you asked a question the operator must answer, or nothing above fits | `input` |

`pre-scope` and `discussing` are two states, not one. `pre-scope` means no task exists yet. `discussing` means a task exists and its proposal is under discussion. An unknown value renders as the `other` mark. The empty string removes the label; you rarely need it. See [[dev-knw-strk-sessions]] for the contract the TUI reads.

## Closing a Strike Session

A session accumulates targets. The operator links another row to a live session with `o i` at any time, and Strike has no reverse channel to say so. The bindings sidecar is therefore the only honest answer to "which targets is this session working".

`strike session finish <sessionId>` reads that sidecar and finishes the whole set. Run [[dev-sk-strk-close]] to end a session: it shows the operator the list, finishes what can be finished, reports what cannot, and hands off to the Orbh close skill.

## The Sync Log

Strike writes each sync event to its Foundation event log ((Task) 311). When the operator reports a sync fault (offline, `SYNC FAILED`, a row that does not arrive, a command that does not apply), read the log before you read code or try to reproduce the fault:

```bash
flint foundation logs --service strike --role tui --event sync --since 30m
```

The log holds ids, codes, revisions, counts, and durations, never a title or a token. Read [[knw-strk-sync_log]] for the commands, where to look first for each fault, and the event names.

## Rules

- Strike declares no Mesh artifact type. Task artifacts belong to the Projects shard.
- Ids in the preamble are opaque strings. Quote them; never construct or guess one.
- Every row has a subsection after (Task) 215, so the subsection clause is always present in a new prompt. Old prompts may still lack it. Tolerate the missing clause instead of failing.
- Strike state is machine-local. Do not edit `~/.nuucognition/strike-local.json` or the bindings sidecar by hand.

# Headless

A headless or subagent Orbh session has no person at the terminal. It loads this init with `flint shard start strk`, as an interactive session does. Follow the `> [!headless]` callouts of the workflows.

You are a HEADLESS Orbh session launched from Strike. No operator watches a terminal pane for you. Your channel to the operator is the **turn result**: Strike renders your session as a chat built from the launch prompt, the messages Strike sends you, and the results you return. Write every result for that chat.

The model knowledge of this init comes through its required reading. The rules below replace the interactive session rules.

## The Turn Contract

1. Read your first prompt. It carries the Strike preamble (task title, target id, subsection, ancestry), a workflow instruction, and the operator's extra text. The contract is in [[knw-strk-sessions]].
2. Do one coherent unit of work per turn.
3. End every turn with an explicit return:

```bash
flint orbh session return --await "<result>"    # you expect a reply
flint orbh session return --finish "<result>"   # the work is done
```

4. While you await, the operator replies from the Strike chat. The reply arrives as a message; the orchestrator wakes you with it as your next prompt. Treat that text as the operator speaking.

## Result Style

- The result IS the message the operator reads. Write it as a direct reply, in ASD-STE100 style: short sentences, one idea per sentence, active voice.
- Lead with the outcome. Then give the detail that changes what the operator does next.
- Name every artifact you created or changed, with its exact Mesh title.
- End with the question or the next step, when one exists.

## Rules in a Headless Session

- **Never block on input.** Do not run `flint orbh session ask`. Do not run `flint orbh approval request`. Return with `--await` and ask in the result instead.
- **Set the lifecycle label before every await.** The mirror table of § The Lifecycle Label above holds: `pre-scope` before a task exists, `scoped` when a scoped task waits for go, `discussing` during a proposal conversation, `review` when finished work waits for review, `input` when you are blocked.

```bash
flint orbh session set strike-lifecycle <value>
```

- **Record the Strike ids** (`strike-target-id`, `strike-subsection-id`, `strike-ancestry`, and for a tied row `strike-ops-task-id` and `strike-ops-org`) exactly as the workflows do.
- **Read the Operations task of a tied row.** When the task phrase of the preamble carries `Operations task id: <id>`, the Strike row is tied to that Operations task. Before other work, in every launch mode, read the full context of the task. Run `ops task context <id> --org <org>` (omit `--org` when the clause names no org). The command prints the facts of the task, the project description, the task description, and every comment of the Updates timeline, with no Waybill event. Use the description and the comments as context for the work, and carry a short summary of them into any task you scope. When the clause says `data: dev`, the id is a dev id. Start every `ops` command of that task with `OPS_DATA_MODE=dev ndv run`, for example `OPS_DATA_MODE=dev ndv run ops task context <id> --org <org>`. First run `OPS_DATA_MODE=dev ndv run ops mode --json`, and read or write nothing when its `dataMode` is not `dev`: `ndv run` alone selects only the dev launcher, and that launcher still follows an inherited `OPS_DATA_MODE` or the stored data mode ((Task) 341 review). When no command can read the task (no credential, a deleted task, a wrong data mode), say so in one sentence of your next result and continue without the context. The clause contract is in [[knw-strk-sessions]].
- **Work with Operations ((Task) 341).** A tied launch ends its instructions with one Operations sentence that names the workflow [[dev-wkfl-strk-ops_do]] (a launch prompt of the Strike TUI can still name it `hwkfl-strk-ops_do`: see § Old Launch Prompts). Follow that workflow and its `> [!headless]` callouts beside the workflow of the launch mode: it starts the Operations task, posts one place-relative update at each meaningful point (start, blocker, review stop, done), writes only to the linked task, and ends each comment with the agent signature line, which names the fleet and the Orbh session. It never writes to a task found only by place, and it runs `ops task done` only when the operator tells you to.
- **Register the session** with the task title, so every Orbh surface names it.
- Session messages from Strike (`Strike: …`) are events, not operator replies. Read them, update your keys, and continue.
- All artifact conventions (templates, frontmatter, `orbh-sessions`, authors) are unchanged.
- **Read the sync log for a sync fault.** When the work is a Strike sync fault (offline, `SYNC FAILED`, a row that does not arrive, a command that does not apply), read the sync log before you read code: `flint foundation logs --service strike --role tui --event sync --since 30m`. The log holds ids, codes, revisions, counts, and durations, never a title or a token. See [[knw-strk-sync_log]].

## Workflows in a Headless Session

| File | When |
|------|------|
| [[dev-wkfl-strk-strike_start]] | Default launch mode: scope the target as a task, await the go |
| [[dev-wkfl-strk-discuss_start]] | Discuss First: talk through awaited returns, then scope |
| [[dev-wkfl-strk-create_and_do]] | Create and Do: create the task, do the work, return the summary |
| [[dev-wkfl-strk-ops_do]] | Beside any mode, when the row is tied to an Operations task: read it, start it, post its updates |

Follow the `> [!headless]` callouts of each workflow. [[dev-wkfl-strk-create_and_do]] is for a headless session only.

## Old Launch Prompts

The Strike TUI composes the headless launch prompt (`apps/strike-tui/src/orbh/preamble.ts`). It reads the shard folder of the bound Flint at the launch. A prompt can still name `flint shard hstart strk` and an old headless name of a workflow: from an older Strike TUI, from a launch on a peer machine (Strike does not know the shard folder there), or in a session that started before the migration. Since NUU Orbh Task 748, the shard has one workflow for each mode. `flint shard hstart strk` loads this init with a notice. Read each old name as the workflow of the same name, and follow its `> [!headless]` callouts:

| The prompt names | Follow |
|------------------|--------|
| `hwkfl-strk-strike_start` | [[dev-wkfl-strk-strike_start]] |
| `hwkfl-strk-discuss_start` | [[dev-wkfl-strk-discuss_start]] |
| `hwkfl-strk-create_and_do` | [[dev-wkfl-strk-create_and_do]] |
| `hwkfl-strk-ops_do` | [[dev-wkfl-strk-ops_do]] |

## Orbh Interface Keys

Set each key with `flint orbh session set <key> <value>`.

| Key | Description | When to update |
|-----|-------------|----------------|
| `strike-lifecycle` | The lifecycle label of the Strike row | Before every await (see § The Lifecycle Label above) |
| `strike-target-id`, `strike-subsection-id`, `strike-ancestry` | The Strike ids of the preamble | Stage 1 of the launch workflow |
| `strike-ops-task-id`, `strike-ops-org` | The tied Operations task | Stage 1, for a tied row |
| `artifacts` | The task that the session scoped or did | When the task exists |

### Phase Values

The phase of a headless Strike session is its lifecycle label: `pre-scope`, `scoped`, `discussing`, `review`, or `input`. The table of § The Lifecycle Label above gives the meaning of each value and when to set it.

## Result Delivery

End every turn with `flint orbh session return --await "<result>"` when you expect a reply, or `flint orbh session return --finish "<result>"` when the work is done (see § The Turn Contract). The result is the chat message that the operator reads (see § Result Style).
