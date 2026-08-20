---
description: "Finish every Strike target bound to this session, then close the Orbh session"
---

> [!important] THIS FILE IS AN INSTRUCTION. WHEN REFERENCED IT IS MEANT TO BE TAKEN AS AN ACTION.

Run `flint shard start strk` if you haven't already.

# Skill: Strike Close

End a Strike-launched session cleanly. Finish the Strike targets this session was working, report what happened to each one, then close the Orbh session.

Use this when the work is done and the operator wants the session wrapped up. Do not use it to close a session whose work is unfinished — a refused target is a signal to stop, not to push past.

## Why a command, not a memory

A session works more targets than it launched with. The operator links another row to a live session with `W i` at any time, and Strike has no reverse channel to tell you. So the set of targets is only knowable from Strike's bindings sidecar, never from your own memory of the session.

`strike session finish` reads that sidecar. Trust it over your recollection.

# Input

- Your Orbh session id, in `ORBH_SESSION_ID`
- (Optional) The operator's instruction to close

# Actions

## 1. See what would close

Never finish targets before showing the operator the list.

```bash
strike session finish "$ORBH_SESSION_ID" --dry-run
```

Read the output. Each line names one target and its planned outcome. Handle these cases before going further:

- **No targets bound.** Say so. This session finished no Strike work, which is normal for a session that only scoped a task. Skip to step 4.
- **A target you did not work.** Someone linked a row you never touched. Ask the operator whether to include it. Do not silently finish another agent's work.
- **A target whose work is unfinished.** Stop. Tell the operator what is left. Closing is their call, not yours.

## 2. Finish the targets

```bash
strike session finish "$ORBH_SESSION_ID"
```

Read every line of the report. The outcomes mean:

| Outcome | Meaning | What to do |
|---------|---------|------------|
| `done` | The target is now done | Nothing |
| `was done` | It was already done | Nothing |
| `skipped` | It cannot be finished this way; the line says why | Read the reason — see below |
| `refused` | The engine refused; the line carries its message | Report it to the operator |

The command exits non-zero when any target refused.

**A refusal is usually correct.** The commonest one names incomplete requirements: the target has open subtasks, so it is not done. Do not work around it. Report the open requirements to the operator and let them decide.

**Two skips are expected and are not problems.** A subsection objective has a derived status, so the engine will not set it directly. A target tied to an Operations task is skipped when no Strike TUI is running, because the change would not reach canon — tell the operator to start Strike if that target matters.

## 3. Record the outcome

Put the result where a later session can read it:

```bash
flint orbh update "Finished N Strike target(s) on close: <titles>." --kind completed
```

If any target refused or was skipped, name it in that update. A close that left work behind must say so.

## 4. Close the session

Hand off to [[sk-foh-close]] from the Orbh shard and follow it exactly. It owns the Mesh summary, the final turn return, and the session teardown. Load `Shards/Orbh/init-foh.md` first, as that skill requires.

Its close call ends three things at once: the session record, the harness, and the Obsidian terminal tab this session runs in. Expect the tab to disappear. Nothing runs after it.

Carry into the summary:

- Which Strike targets closed, by title
- Any target that refused or was skipped, with its reason
- The Mesh task artifacts this session produced

Nothing runs after the close call.

# Output

- Every finishable Strike target for this session is `done`
- The operator has seen one line per target, including everything that did not close
- The session's Mesh summary names the closed targets
- The Orbh session is closed, the harness is terminated, and the Obsidian terminal tab is closed

# Notes

- Do not clear `strike-lifecycle` on close. The Strike row keeps the last label on a closed session, so the operator can still see how the work ended.
- The command works whether or not Strike is running. With a live Strike it hands each request to that process, so the change lands through the same engine as a keystroke and the rows update on screen. With no Strike running it writes the store directly.
- One Strike instance runs per machine. If the command reports that Strike is already running when you believe it is not, the previous instance's lock is stale and the next launch reclaims it — this is not an error you need to fix.
