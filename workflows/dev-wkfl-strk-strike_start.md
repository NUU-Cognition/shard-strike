---
description: "Orient a Strike-launched session in its Strike context, then scope the named target as a task"
orbh-sessions:
  - "[[acefe761-97a3-4421-86ee-08717b7cb565]]"
  - "[[3fb6e594-a179-4c0b-ba67-833b2ccaea67]]"
  - "[[61b5f523-537e-425e-ab9a-002b3c2dac39]]"
  - "[[d98d16ae-1388-4228-8610-6954998a395a]]"
---

> [!important] THIS FILE IS AN INSTRUCTION. WHEN REFERENCED IT IS MEANT TO BE TAKEN AS AN ACTION.

Run `flint shard start strk` if you haven't already.

# Workflow: Strike Start

Read the Strike launch preamble, load the Strike context it names, and immediately scope that target as a task with [[wkfl-proj-scope_task]].

This is the first thing a Strike-launched session does. It converts one Strike row into a scoped Mesh task with a written proposal, so the operator gets a reviewable spec instead of an agent guessing at a title.

[[dev-wkfl-strk-discuss_start]] enters this workflow at Stage 2. That workflow already read the preamble and set the session keys, and it carries a discussion record into Stage 3. Skip Stage 1 when you arrive that way.

> [!headless]
> In a headless or subagent Orbh session launched from Strike, the job is the same: convert one Strike row into a scoped Mesh task with a written proposal. But the operator reads your turn results in the Strike chat instead of a terminal pane. No stage waits in the chat; every wait is a `return --await`. Follow the `> [!headless]` callout of each stage. The rules are in the section `# Headless` of [[dev-init-strk]].

# Input

- The composed launch prompt, already in context — it is this session's first prompt
- (Optional) The operator's extra text at the end of that prompt
- (Optional) Anything the operator says after launch

> [!headless]
> - Wake messages from the operator on later turns

# Actions

## Stage 1: Read the Preamble

Parse this session's first prompt. The contract is in [[dev-knw-strk-sessions]]. Extract:

- The task title and the Strike target id
- The subsection title and the Operations subsection id, when the clause is present
- The ancestry — the path of parent rows, subsection first and the task last — when the clause is present
- The Operations task id and its org, when the task phrase carries an Operations clause (`Operations task id: <id>, org: <org>`), and `data: dev` when the clause says so
- The operator's extra text, when present — everything after the strike_start instruction

Record what you found:

```bash
flint orbh session set strike-target-id "<target id>"
flint orbh session set strike-subsection-id "<subsection id>"
flint orbh session set strike-ancestry "<subsection> > <ancestor> > <task>"
flint orbh session set strike-ops-task-id "<Operations task id>"
flint orbh session set strike-ops-org "<Operations org>"
```

Omit the subsection key when the clause is absent, the ancestry key when that clause is absent, and the two Operations keys when the task phrase has no Operations clause. Omit `strike-ops-org` when the clause names no org. Do not invent an id.

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

> [!headless]
> - Re-register with the headless description: `flint orbh session register "<task title>" "Headless: scoping the Strike target as a task."`
> - **No preamble.** Do not ask in a pane. Return at once:
>
>   ```bash
>   flint orbh session set strike-lifecycle input
>   flint orbh session return --await "This session was not launched from Strike. Tell me what to work on."
>   ```

## Stage 2: Establish Strike Context

Confirm your Strike knowledge is loaded. If you have not read them, read [[dev-knw-strk-strike]] and [[dev-knw-strk-sessions]] now.

When Stage 1 found an Operations task, read its full context now, by the rule in the init:

```bash
ops task context <Operations task id> --org <Operations org>
```

Keep the description and the comments for Stage 3. They are the operator's own words about this work, so they outrank a guess from the title. When the command fails, tell the operator in one sentence and continue.

Then gather what already exists for this work:

- Search `Mesh/` for artifacts naming the task title, the subsection title, or any ancestor title
- Search `Mesh/Types/Tasks/` for a task that already covers this row — a relaunch on the same target is normal, and a duplicate task is not wanted
- Read the subsection's own artifacts when the Mesh holds any

If an existing task already covers the row, stop and present it to the operator. Ask whether to continue that task with [[wkfl-proj-do_task]] instead of scoping a new one. Do not create a second task for the same row without the operator saying so.

Once you have a picture of prior work and no duplicate blocks you, progress to the next stage.

> [!headless]
> - When the `ops task context` command fails, say so in one sentence of your next result, and continue.
> - If an existing task covers the row, do not scope a duplicate. Set the lifecycle label to `input` and `return --await` with the existing task's name and the question: continue it, or scope a new one?

## Stage 3: Scope the Task

Run [[wkfl-proj-scope_task]] with the Strike context as its input. Load `Shards/Projects/init-proj.md` first, as that workflow requires.

Carry the Strike context into it:

- The problem to scope is the task title from the preamble, plus the operator's extra text
- Set `from` to the subsection's Mesh artifact when one exists; otherwise leave it blank
- Name the Strike target id, the subsection id, and the ancestry in the task's **Notes**, so a later session can walk back to the Strike row
- Include the launch context in the task's **Context** section: this task came from a Strike launch on that row, under those parents
- When Stage 2 read an Operations task context, summarise the task description and the comments in the task's **Context**, let them shape the requirements, and name the Operations task id, its org, and its URL in the **Notes**. Also set the frontmatter fields `ops-task` and `ops-org` of the task ([[dev-wkfl-strk-ops_do]])

Follow every stage of that workflow, including its proposal review with the operator. It owns the task template, the numbering, and the file naming. When you present the first proposal and stop to wait for the operator, the task exists in `todo`. Set the lifecycle label to `scoped` first:

```bash
flint orbh session set strike-lifecycle scoped
```

Tell the operator, in those words, to say **go** to start the next step. Name [[wkfl-proj-do_task]] as that next step. Use that word. Do not ask for yes.

If the operator asks for changes and you stop again during the proposal conversation, set the lifecycle label to `discussing`. After a later wait in that conversation, tell the operator to say **go** again.

If the operator says **go**, treat the spec as confirmed and progress to the next stage. If the operator confirms the spec without that word, progress to the next stage.

> [!headless]
> Follow the research and creation stages of [[wkfl-proj-scope_task]], but **skip every chat wait** — the proposal review happens through the returned result. When the task exists in `todo` with its proposal written:
>
> ```bash
> flint orbh session set artifacts "(Task) NNN <Name>"
> flint orbh update "Scoped (Task) NNN <Name> from the Strike target." --kind completed
> flint orbh session register "<task title>" "Task NNN scoped; awaiting go."
> flint orbh session set strike-lifecycle scoped
> flint orbh session return --await "<the proposal summary>"
> ```
>
> The returned result carries: the task name, the problem in two or three sentences, the proposed approach, the requirement list in short form, and the closing line: `Say go to start the work.`

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

If the operator said **go**, run [[wkfl-proj-do_task]] now. Do not wait again. When the row is tied to an Operations task, follow [[dev-wkfl-strk-ops_do]] beside it: the go is the start of the Operations task, and the blocker, the review stop, and the done each get one update.

If the operator did not say go, stop here — the task stands on its own. Do not offer the next step a second time.

Inside [[wkfl-proj-do_task]], keep the label true by the mirror table in the init: set it when you stop and wait, never during the work

> [!headless]
> Stage 3 already recorded the task, re-registered the session, and set the label: do not do it again. The next wake carries the operator's reply.
> - **go** (or a clear confirmation): run [[wkfl-proj-do_task]] on the task. When the row is tied to an Operations task, follow [[dev-wkfl-strk-ops_do]] and its `> [!headless]` callouts beside it: the go is the start of the Operations task, and the blocker and the review stop each get one update. Work until done or blocked. Then set the lifecycle label (`review` when finished, `input` when blocked) and `return --await` with the completion summary or the blocker.
> - **Change requests**: apply them to the task, set the label to `discussing`, and `return --await` with the updated proposal.
> - **Stop**: set the task to the state the operator names, and `return --finish` with a one-line close.

# Output

- A `todo` task in `Mesh/Types/Tasks/`, with a written proposal in its Notes
- The Strike target id and the subsection id recorded in that task and on the session interface
- Session title and description naming the Strike task
- The operator holding a reviewed spec, told to say **go** to start the next step

> [!headless]
> - The proposal delivered as a turn result in the Strike chat
> - The session awaiting the operator's go
