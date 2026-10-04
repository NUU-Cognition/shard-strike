---
description: "Headless Do With Operations — read the context of the tied Operations task, start it, post one place-relative update at each meaningful point, and put each Operations question in the returned result"
orbh-sessions:
  - "[[61b5f523-537e-425e-ab9a-002b3c2dac39]]"
---

> [!important] THIS FILE IS AN INSTRUCTION. WHEN REFERENCED IT IS MEANT TO BE TAKEN AS AN ACTION.

Run `flint shard hstart strk` if you haven't already.

# Headless Workflow: Do With Operations

The headless form of [[dev-wkfl-strk-ops_do]]. It runs beside the headless workflow of the launch mode ([[dev-hwkfl-strk-strike_start]], [[dev-hwkfl-strk-discuss_start]], or [[dev-hwkfl-strk-create_and_do]]). That workflow does the work. This workflow reads the tied Operations task, starts it, and posts few updates. No operator watches a pane, so each question goes into the returned result, and a write that needs the operator's yes waits for a wake.

The work is tied to an Operations task when the Strike preamble carries an Operations clause, when the Mesh task of the work has the field `ops-task`, or when the operator's message names an Operations task id or URL.

# Rules

Obey the eight rules of [[dev-wkfl-strk-ops_do]]: write only to the linked task, read before you write, post few comments, sign each comment, check before you repeat a write, respect the data mode, respect the derived state, and write in Simplified Technical English. Two rules are stricter here:

- **No write by place alone.** A task found only by the place of this Flint is a candidate. Never start it and never comment on it. Put the candidates in the result and ask the operator to name one.
- **No `ops task done` without an order.** Run it only when the launch prompt or a wake message tells you to close the Operations task.

# Actions

## Stage 1: Find the Operations Task

Use the order of Stage 1 of [[dev-wkfl-strk-ops_do]]: the Strike preamble, the `ops-task` field of the Mesh task, an id or URL in the operator's message, then the place of this Flint.

When only the place names candidates, return them and wait:

```bash
flint orbh session set strike-lifecycle input
flint orbh session return --await "<the candidates: title, place, URL; ask which task the work belongs to>"
```

Record the link on the session (`strike-ops-task-id`, `strike-ops-org`) and in the frontmatter of the Mesh task (`ops-task`, `ops-org`), as the interactive workflow does.

## Stage 2: Read the Context and the Place

Run `ops task context <id> --org <org>` for the task, and `flint org show --json` for the Flint name and address. Keep the status, the place, the URL, the description, and the comments.

When the read fails, use the table of Stage 2 of [[dev-wkfl-strk-ops_do]]. Say the failure in one sentence of your next result. When the status is `done`, do not reopen the task: say so in the result, and continue only when the operator answers.

## Stage 3: Start

The start is the start of the implementation: in [[dev-hwkfl-strk-strike_start]], the wake that says **go**; in [[dev-hwkfl-strk-create_and_do]], Stage 2. At the start, run `ops task start <id> --org <org>` when the status is `todo`, then post the start update with the format of [[dev-wkfl-strk-ops_do]].

## Stage 4: Work

Post no comment while you work. Post the blocker update when the Mesh task goes to `blocked`, before the `return --await` that names the blocker. Post the review update when the Mesh task goes to `review`, before the `return --await` with the completion summary. In that result, say that `ops task done <id>` waits for the operator.

## Stage 5: Done

When a wake message tells you to close the Operations task, run `ops task done <id> --org <org>`, then post the done update. Name the result of both commands in the returned result.

# Output

- The Operations task started, with at most one update for each meaningful point
- Each Operations question and each candidate list in a returned result
- No write outside the linked task
