---
description: "Orient a Strike-launched session in its Strike context, then scope the named target as a task"
orbh-sessions:
  - "[[acefe761-97a3-4421-86ee-08717b7cb565]]"
---

> [!important] THIS FILE IS AN INSTRUCTION. WHEN REFERENCED IT IS MEANT TO BE TAKEN AS AN ACTION.

Run `flint shard start strk` if you haven't already.

# Workflow: Strike Start

Read the Strike launch preamble, load the Strike context it names, and immediately scope that target as a task with [[wkfl-proj-scope_task]].

This is the first thing a Strike-launched session does. It converts one Strike row into a scoped Mesh task with a written proposal, so the operator gets a reviewable spec instead of an agent guessing at a title.

[[dev-wkfl-strk-discuss_start]] enters this workflow at Stage 2. That workflow already read the preamble and set the session keys, and it carries a discussion record into Stage 3. Skip Stage 1 when you arrive that way.

# Input

- The composed launch prompt, already in context — it is this session's first prompt
- (Optional) The operator's extra text at the end of that prompt
- (Optional) Anything the operator says after launch

# Actions

## Stage 1: Read the Preamble

Parse this session's first prompt. The contract is in [[dev-knw-strk-sessions]]. Extract:

- The task title and the Strike target id
- The subsection title and the Operations subsection id, when the clause is present
- The ancestry — the path of parent rows, subsection first and the task last — when the clause is present
- The operator's extra text, when present — everything after the strike_start instruction

Record what you found:

```bash
flint orbh session set strike-target-id "<target id>"
flint orbh session set strike-subsection-id "<subsection id>"
flint orbh session set strike-ancestry "<subsection> > <ancestor> > <task>"
```

Omit the subsection key when the clause is absent, and the ancestry key when that clause is absent. Do not invent an id.

Read the ancestry as work context, not as decoration. It tells you what the target hangs under, so the scoped task states the right problem instead of restating a leaf title.

Re-register so every Orbh surface shows the Strike task:

```bash
flint orbh session register "<task title>" "Scoping the Strike target as a task."
```

Handle the two degenerate cases:

- **No preamble.** This session was not launched from Strike. Say so, stop this workflow, and ask the operator what they want.
- **No subsection clause.** The Strike row is local-only. Continue — the task simply has no subsection context.
- **No ancestry clause.** The row hangs under nothing. Continue with the task title alone.

Once you hold the title and the ids, progress to the next stage.

## Stage 2: Establish Strike Context

Confirm your Strike knowledge is loaded. If you have not read them, read [[dev-knw-strk-strike]] and [[dev-knw-strk-sessions]] now.

Then gather what already exists for this work:

- Search `Mesh/` for artifacts naming the task title, the subsection title, or any ancestor title
- Search `Mesh/Types/Tasks/` for a task that already covers this row — a relaunch on the same target is normal, and a duplicate task is not wanted
- Read the subsection's own artifacts when the Mesh holds any

If an existing task already covers the row, stop and present it to the operator. Ask whether to continue that task with [[wkfl-proj-do_task]] instead of scoping a new one. Do not create a second task for the same row without the operator saying so.

Once you have a picture of prior work and no duplicate blocks you, progress to the next stage.

## Stage 3: Scope the Task

Run [[wkfl-proj-scope_task]] with the Strike context as its input. Load `Shards/Projects/init-proj.md` first, as that workflow requires.

Carry the Strike context into it:

- The problem to scope is the task title from the preamble, plus the operator's extra text
- Set `from` to the subsection's Mesh artifact when one exists; otherwise leave it blank
- Name the Strike target id, the subsection id, and the ancestry in the task's **Notes**, so a later session can walk back to the Strike row
- Include the launch context in the task's **Context** section: this task came from a Strike launch on that row, under those parents

Follow every stage of that workflow, including its proposal review with the operator. It owns the task template, the numbering, and the file naming. When you present the first proposal and stop to wait for the operator, the task exists in `todo`. Set the lifecycle label to `scoped` first:

```bash
flint orbh session set strike-lifecycle scoped
```

Tell the operator, in those words, to say **go** to start the next step. Name [[wkfl-proj-do_task]] as that next step. Use that word. Do not ask for yes.

If the operator asks for changes and you stop again during the proposal conversation, set the lifecycle label to `discussing`. After a later wait in that conversation, tell the operator to say **go** again.

If the operator says **go**, treat the spec as confirmed and progress to the next stage. If the operator confirms the spec without that word, progress to the next stage.

## Stage 4: Hand Off

- Record the created task on the session:

  ```bash
  flint orbh session set artifacts "(Task) NNN <Name>"
  flint orbh update "Scoped (Task) NNN <Name> from the Strike target." --kind completed
  ```

- Re-register with the scope now settled:

  ```bash
  flint orbh session register "<task title>" "Task NNN scoped and ready to work."
  ```

- Set the lifecycle label, so the Strike row shows the task waits for a go:

  ```bash
  flint orbh session set strike-lifecycle scoped
  ```

If the operator said **go**, run [[wkfl-proj-do_task]] now. Do not wait again.

If the operator did not say go, stop here — the task stands on its own. Do not offer the next step a second time.

Inside [[wkfl-proj-do_task]], keep the label true by the mirror table in the init: set it when you stop and wait, never during the work

# Output

- A `todo` task in `Mesh/Types/Tasks/`, with a written proposal in its Notes
- The Strike target id and the subsection id recorded in that task and on the session interface
- Session title and description naming the Strike task
- The operator holding a reviewed spec, told to say **go** to start the next step
