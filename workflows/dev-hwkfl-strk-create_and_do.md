---
description: "Headless Create and Do — create the task, work it to completion, and return the completion summary; review rides the returned result"
---

> [!important] THIS FILE IS AN INSTRUCTION. WHEN REFERENCED IT IS MEANT TO BE TAKEN AS AN ACTION.

Run `flint shard hstart strk` if you haven't already.

# Headless Workflow: Create and Do

The headless wrapper of [[wkfl-proj-create_and_do_task]]. Create the task, execute it, and deliver the completion summary as the turn result. The operator's review happens in the Strike chat, not in a terminal pane.

# Input

- The composed launch prompt, already in context
- (Optional) The operator's extra text — instructions for the work
- Wake messages from the operator on later turns

# Actions

## Stage 1: Read the Preamble and Register

Same as Stage 1 of [[hwkfl-strk-strike_start]]: record the ids, then register:

```bash
flint orbh session register "<task title>" "Headless: creating and doing the Strike task."
```

## Stage 2: Create and Work the Task

Run [[wkfl-proj-create_and_do_task]] Stages 1 and 2. Load `Shards/Projects/init-proj.md` first.

- Record the Strike target id and the subsection id in the task's Notes
- Tick checkboxes as you complete them; keep the Task Log current
- Commit WIP by [[knw-proj-wip_commits]] when the task edits a repo

If you hit a blocker only the operator can clear: set the task to `blocked`, log the reason, then

```bash
flint orbh session set strike-lifecycle input
flint orbh session return --await "<the blocker and the decision you need>"
```

Continue when the wake answers it.

## Stage 3: Return for Review

When the work is complete, set the task to `review` and return:

```bash
flint orbh session set artifacts "(Task) NNN <Name>"
flint orbh update "Completed (Task) NNN <Name>." --kind completed
flint orbh session set strike-lifecycle review
flint orbh session return --await "<the completion summary>"
```

The result carries: what was built or changed, how it was verified, every artifact and commit by exact name, and anything left open.

## Stage 4: Close

- **Change requests** in the next wake: apply them, keep the task in `review`, and `return --await` with the delta.
- **Approval**: set the task to `done` with the completion date, and `return --finish` with a one-line close.

# Output

- A completed task in `Mesh/Types/Tasks/`, reviewed through the Strike chat
- The completion summary delivered as a turn result
