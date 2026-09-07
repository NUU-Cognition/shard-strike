---
description: "Headless Discuss First — discuss the launched target through awaited turn results, then hand off to the headless Strike Start"
---

> [!important] THIS FILE IS AN INSTRUCTION. WHEN REFERENCED IT IS MEANT TO BE TAKEN AS AN ACTION.

Run `flint shard hstart strk` if you haven't already.

# Headless Workflow: Discuss First

The headless form of [[wkfl-strk-discuss_start]]. The discussion happens in the Strike chat: each of your turns returns one discussion reply and awaits, and each wake carries the operator's next message. Write no Mesh artifact until the operator agrees to scope.

# Input

- The composed launch prompt, already in context
- Wake messages from the operator on later turns

# Actions

## Stage 1: Read the Preamble

Same as Stage 1 of [[hwkfl-strk-strike_start]]: record the ids, then register:

```bash
flint orbh session register "<task title>" "Headless: discussing the Strike target before scope."
```

## Stage 2: Gather Light Context

One short pass, as the interactive form rules: search for an existing task on this row, read the obvious artifact or code the title points at. If an existing task covers the row, say so in your first returned reply and ask whether to continue it instead.

## Stage 3: Discuss Through Returns

Open the discussion with your first returned result. It carries:

- What you understand the row to mean
- What you are unsure about
- The questions that change the shape of the work

Before every await in this stage:

```bash
flint orbh session set strike-lifecycle pre-scope
flint orbh session return --await "<your reply>"
```

Keep a running discussion record in your context across turns: the problem in the operator's words, every option weighed, every decision, every constraint, every open question. Each wake is one operator message; answer it and await again.

## Stage 4: Offer the Handoff

When the discussion settles, return one summary result: what you would scope, the chosen approach, the open questions, and the closing line: `Say go to scope this as a task.` Set the label to `pre-scope` and await.

## Stage 5: Hand Off

On the operator's go, run [[hwkfl-strk-strike_start]] from its Stage 3. Carry the discussion record into the scope, and write the **Pre-Scope Discussion** heading into the task's Notes exactly as [[wkfl-strk-discuss_start]] Stage 5 defines it. From this point that workflow owns the labels: `scoped` on the proposal await, `discussing` in the proposal conversation.

If the operator stops without a task, `return --finish` with a one-line close — the discussion was the work.

# Output

- A discussion held in the Strike chat, one turn result per exchange
- On a go: a `todo` task carrying the discussion summary, scoped by the headless Strike Start
- No Mesh artifact before the operator agreed
