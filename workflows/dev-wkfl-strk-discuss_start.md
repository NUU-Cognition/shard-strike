---
description: "Discuss a Strike-launched target with the operator in the chat, then wrap strike_start and carry the discussion into the scoped task"
orbh-sessions:
  - "[[07edd93f-9dcc-4ad7-9b50-791dd1c33fdc]]"
---

> [!important] THIS FILE IS AN INSTRUCTION. WHEN REFERENCED IT IS MEANT TO BE TAKEN AS AN ACTION.

Run `flint shard start strk` if you haven't already.

# Workflow: Discuss First

Read the Strike launch preamble, discuss the target with the operator in the chat, and only then hand off to [[dev-wkfl-strk-strike_start]]. The scoped task carries a detailed summary of the discussion.

This is the pre-scope sibling of [[dev-wkfl-strk-strike_start]]. That workflow scopes at once, so the operator's first sight of the session is a written task. This one talks first. It writes no Mesh artifact until the operator says go.

**Write nothing to the Mesh before Stage 5.** No task, no notepad, no note. The discussion lives in the chat. The scoped task is the only durable output, and the operator asks for it.

# Input

- The composed launch prompt, already in context — it is this session's first prompt
- (Optional) The operator's extra text at the end of that prompt
- The operator's replies during the discussion

# Actions

## Stage 1: Read the Preamble

Parse this session's first prompt. The contract is in [[dev-knw-strk-sessions]]. Extract:

- The task title and the Strike target id
- The subsection title and the Operations subsection id, when the clause is present
- The ancestry — the path of parent rows, subsection first and the task last — when the clause is present
- The operator's extra text, when present — everything after the discuss_start instruction

Record what you found:

```bash
flint orbh session set strike-target-id "<target id>"
flint orbh session set strike-subsection-id "<subsection id>"
flint orbh session set strike-ancestry "<subsection> > <ancestor> > <task>"
```

Omit the subsection key when the clause is absent, and the ancestry key when that clause is absent. Do not invent an id.

Re-register so every Orbh surface shows the Strike task:

```bash
flint orbh session register "<task title>" "Discussing the Strike target before scope."
```

Handle the two degenerate cases:

- **No preamble.** This session was not launched from Strike. Say so, stop this workflow, and ask the operator what they want.
- **No subsection clause.** The Strike row is local-only. Continue — the discussion simply has no subsection context.
- **No ancestry clause.** The row hangs under nothing. Continue with the task title alone.

Once you hold the title and the ids, progress to the next stage.

## Stage 2: Gather Light Context

Read enough to discuss well. Do not run the full research pass — that belongs to [[wkfl-proj-scope_task]] in Stage 5, and doing it twice wastes the session.

- Search `Mesh/Types/Tasks/` for a task that already covers this row
- Search `Mesh/` for artifacts naming the task title, the subsection title, or any ancestor title
- Read the ancestry as work context. It tells you what the target hangs under
- Open the obvious code or artifact the title points at, when one exists

If an existing task already covers the row, say so before you discuss. Ask whether to continue that task with [[wkfl-proj-do_task]] instead. Do not open a discussion that ends in a duplicate task.

Keep this stage short. One pass is enough.

Once you can state the problem in your own words, progress to the next stage.

## Stage 3: Discuss

Talk with the operator in the chat. This stage is the point of the workflow.

- Open with what you understand the row to mean, and say what you are unsure about
- Ask the questions that change the shape of the work, not the questions that change a detail
- Offer readings of the row when it is ambiguous, and name the trade-off in each
- Say when you disagree, and say why
- Let the operator lead. The discussion ends when they are satisfied, not when you are

**Keep a running discussion record.** Hold it in your own context, and update it after every exchange. It carries:

- The problem as the operator states it, in their words
- Every option weighed, and the reason each one was kept or dropped
- Every decision the operator made, and what it rules out
- Every constraint the operator named
- Every question still open

Set the lifecycle label before every wait in this stage:

```bash
flint orbh session set strike-lifecycle pre-scope
```

This value means "no task exists yet, and we are still deciding what it should be". It is not `discussing`, which means "a task exists and its proposal is under discussion". The Strike row shows a different glyph for each, so the operator can tell the two phases apart.

Once the discussion settles, progress to the next stage.

## Stage 4: Offer the Handoff

Do not scope on your own. The operator decides when the talk is finished.

- Say that the discussion looks settled
- State in a few sentences what you would scope, and name the approach the discussion chose
- Name any question the discussion left open
- Ask whether to run [[dev-wkfl-strk-strike_start]] now

Set the lifecycle label to `pre-scope` again and wait.

If the operator wants to keep talking, return to Stage 3. If the operator wants to stop without a task, stop here — the discussion was the work, and no artifact is owed.

Once the operator agrees to scope, progress to the next stage.

## Stage 5: Hand Off to Strike Start

Run [[dev-wkfl-strk-strike_start]] from its Stage 2. Its Stage 1 is already done: you read the preamble and set the session keys in Stage 1 of this workflow.

Carry the discussion record into it:

- The problem to scope is what the discussion decided, not the raw row title
- Pass the chosen approach into [[wkfl-proj-scope_task]] as its research input. Its Stage 1 research now confirms and deepens a chosen direction instead of casting a wide net
- Let its Stage 2 proposal state the approach the discussion chose

**The scoped task's Notes must carry a detailed summary of the discussion**, under its own heading:

```markdown
## Pre-Scope Discussion

[The problem as the operator stated it.]

[Every option weighed, and why each was kept or dropped.]

[Every decision the operator made, and what it ruled out.]

[Every constraint the operator named.]
```

Write it in paragraphs, as the rest of the Notes section is written. Write it in detail. A later session must be able to read it and understand why the scope has this shape, without the chat.

Follow every remaining stage of [[dev-wkfl-strk-strike_start]], including the proposal review. From this point the lifecycle mirror table in the Strike init file governs the label: the first wait after scoping uses `scoped`, and a proposal conversation uses `discussing`. The `pre-scope` value belongs to this workflow only, and you do not return to it.

# Output

- A `todo` task in `Mesh/Types/Tasks/`, scoped from a discussion instead of from a bare row title
- A **Pre-Scope Discussion** heading in that task's Notes, holding a detailed summary of the chat
- The Strike target id and the subsection id recorded in that task and on the session interface
- No Mesh artifact written before the operator agreed to scope
