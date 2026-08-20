---
required-reading:
  - "[[dev-knw-strk-strike]]"
  - "[[dev-knw-strk-sessions]]"
orbh-sessions:
  - "[[c174bf33-b199-4688-b89b-f26247582f78]]"
  - "[[acefe761-97a3-4421-86ee-08717b7cb565]]"
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
| `input` | confused robot | The session is blocked, needs a decision, or is in any other state |

**Set the label when you finish a piece of work and are about to wait for the operator. Never set it during the work.** The TUI hides the label while the session works and shows the last value in every rest state, including a closed session. So the label you set as your last action before you wait is the one the operator sees. Do not clear it on close.

The mirror table below maps the state you leave the work in to the label. It holds for the whole session, also inside workflows owned by other shards (do_task, review_task, notepad start and continue).

| You stop and wait because | Set |
|---------------------------|-----|
| You wait during a pre-scope discussion, before any task exists | `pre-scope` |
| The task is `todo` after scoping, including the first proposal wait | `scoped` |
| You continue a proposal conversation, reply in a notepad, or discuss a design | `discussing` |
| The task is `review`, `reviewing`, or `reviewed` | `review` |
| The task is `blocked`, you asked a question the operator must answer, or nothing above fits | `input` |

`pre-scope` and `discussing` are two states, not one. `pre-scope` means no task exists yet. `discussing` means a task exists and its proposal is under discussion. An unknown value renders as the confused robot. The empty string removes the label; you rarely need it. See [[dev-knw-strk-sessions]] for the contract the TUI reads.

## Closing a Strike Session

A session accumulates targets. The operator links another row to a live session with `o i` at any time, and Strike has no reverse channel to say so. The bindings sidecar is therefore the only honest answer to "which targets is this session working".

`strike session finish <sessionId>` reads that sidecar and finishes the whole set. Run [[dev-sk-strk-close]] to end a session: it shows the operator the list, finishes what can be finished, reports what cannot, and hands off to the Orbh close skill.

## Rules

- Strike declares no Mesh artifact type. Task artifacts belong to the Projects shard.
- Ids in the preamble are opaque strings. Quote them; never construct or guess one.
- Every row has a subsection after (Task) 215, so the subsection clause is always present in a new prompt. Old prompts may still lack it. Tolerate the missing clause instead of failing.
- Strike state is machine-local. Do not edit `~/.nuucognition/strike-local.json` or the bindings sidecar by hand.
