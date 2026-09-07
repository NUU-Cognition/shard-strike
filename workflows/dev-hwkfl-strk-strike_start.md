---
description: "Headless Strike Start — scope the launched target as a task, return the proposal as the turn result, and await the go"
---

> [!important] THIS FILE IS AN INSTRUCTION. WHEN REFERENCED IT IS MEANT TO BE TAKEN AS AN ACTION.

Run `flint shard hstart strk` if you haven't already.

# Headless Workflow: Strike Start

The headless form of [[wkfl-strk-strike_start]]. Same job — convert one Strike row into a scoped Mesh task with a written proposal — but the operator reads your turn results in the Strike chat instead of a terminal pane. No stage waits in the chat; every wait is a `return --await`.

# Input

- The composed launch prompt, already in context
- (Optional) The operator's extra text at the end of that prompt
- Wake messages from the operator on later turns

# Actions

## Stage 1: Read the Preamble

Parse the first prompt by the contract in [[knw-strk-sessions]]. Record the ids and re-register:

```bash
flint orbh session set strike-target-id "<target id>"
flint orbh session set strike-subsection-id "<subsection id>"
flint orbh session set strike-ancestry "<subsection> > <ancestor> > <task>"
flint orbh session register "<task title>" "Headless: scoping the Strike target as a task."
```

Omit a key when its clause is absent. Never invent an id.

If the prompt carries no Strike preamble, return at once:

```bash
flint orbh session set strike-lifecycle input
flint orbh session return --await "This session was not launched from Strike. Tell me what to work on."
```

## Stage 2: Establish Context

- Search `Mesh/Types/Tasks/` for a task that already covers this row.
- Search `Mesh/` for artifacts naming the task title, the subsection title, or an ancestor title.

If an existing task covers the row, do not scope a duplicate. Set the lifecycle label to `input` and `return --await` with the existing task's name and the question: continue it, or scope a new one?

## Stage 3: Scope the Task

Run [[wkfl-proj-scope_task]] with the Strike context as input. Load `Shards/Projects/init-proj.md` first. Follow its research and creation stages, but **skip every chat wait** — the proposal review happens through the returned result.

- Name the Strike target id, the subsection id, and the ancestry in the task's Notes
- State the launch context in the task's Context section

When the task exists in `todo` with its proposal written:

```bash
flint orbh session set artifacts "(Task) NNN <Name>"
flint orbh update "Scoped (Task) NNN <Name> from the Strike target." --kind completed
flint orbh session register "<task title>" "Task NNN scoped; awaiting go."
flint orbh session set strike-lifecycle scoped
flint orbh session return --await "<the proposal summary>"
```

The returned result carries: the task name, the problem in two or three sentences, the proposed approach, the requirement list in short form, and the closing line: `Say go to start the work.`

## Stage 4: The Go

The next wake carries the operator's reply.

- **go** (or a clear confirmation): run [[wkfl-proj-do_task]] on the task. Work until done or blocked. Then set the lifecycle label (`review` when finished, `input` when blocked) and `return --await` with the completion summary or the blocker.
- **Change requests**: apply them to the task, set the label to `discussing`, and `return --await` with the updated proposal.
- **Stop**: set the task to the state the operator names, and `return --finish` with a one-line close.

# Output

- A `todo` task in `Mesh/Types/Tasks/` with a written proposal, exactly as the interactive form produces
- The proposal delivered as a turn result in the Strike chat
- The session awaiting the operator's go
