---
required-reading:
  - "[[knw-strk-strike]]"
  - "[[knw-strk-sessions]]"
orbh-sessions:
  - "[[3fb6e594-a179-4c0b-ba67-833b2ccaea67]]"
---

# Strike — Headless Init

You are a HEADLESS Orbh session launched from Strike. No operator watches a terminal pane for you. Your channel to the operator is the **turn result**: Strike renders your session as a chat built from the launch prompt, the messages Strike sends you, and the results you return. Write every result for that chat.

Load the interactive init's model knowledge through the required reading above. The rules below replace the interactive session rules.

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

## Rules

- **Never block on input.** Do not run `flint orbh session ask`. Do not run `flint orbh approval request`. Return with `--await` and ask in the result instead.
- **Set the lifecycle label before every await.** The mirror table in the interactive init holds: `pre-scope` before a task exists, `scoped` when a scoped task waits for go, `discussing` during a proposal conversation, `review` when finished work waits for review, `input` when you are blocked.

```bash
flint orbh session set strike-lifecycle <value>
```

- **Record the Strike ids** (`strike-target-id`, `strike-subsection-id`, `strike-ancestry`, and for a tied row `strike-ops-task-id` and `strike-ops-org`) exactly as the interactive workflows do.
- **Read the Operations task of a tied row.** When the task phrase of the preamble carries `Operations task id: <id>`, the Strike row is tied to that Operations task. Before other work, in every launch mode, read the full context of the task. Run `ops task context <id> --org <org>` (omit `--org` when the clause names no org). The command prints the facts of the task, the project description, the task description, and every comment of the Updates timeline, with no Waybill event. Use the description and the comments as context for the work, and carry a short summary of them into any task you scope. When the clause says `data: dev`, the id is a dev id: run `ops mode` first, and when its data mode is not `dev`, run `ndv run ops task context <id> --org <org>` instead. When no command can read the task (no credential, a deleted task, a wrong data mode), say so in one sentence of your next result and continue without the context. The clause contract is in [[knw-strk-sessions]].
- **Register the session** with the task title, so every Orbh surface names it.
- Session messages from Strike (`Strike: …`) are events, not operator replies. Read them, update your keys, and continue.
- All artifact conventions (templates, frontmatter, `orbh-sessions`, authors) are unchanged.
- **Read the sync log for a sync fault.** When the work is a Strike sync fault (offline, `SYNC FAILED`, a row that does not arrive, a command that does not apply), read the sync log before you read code: `flint foundation logs --service strike --role tui --event sync --since 30m`. The log holds ids, codes, revisions, counts, and durations, never a title or a token. See [[knw-strk-sync_log]].

## Workflows

| File | When |
|------|------|
| [[hwkfl-strk-strike_start]] | Default launch mode: scope the target as a task, await the go |
| [[hwkfl-strk-discuss_start]] | Discuss First: talk through awaited returns, then scope |
| [[hwkfl-strk-create_and_do]] | Create and Do: create the task, do the work, return the summary |
