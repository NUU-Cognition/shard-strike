---
description: "Work tied to an Operations task — read its context, start it, and post one place-relative update at the start, at a blocker, at a review stop, and at done, through the ops CLI"
orbh-sessions:
  - "[[61b5f523-537e-425e-ab9a-002b3c2dac39]]"
---

> [!important] THIS FILE IS AN INSTRUCTION. WHEN REFERENCED IT IS MEANT TO BE TAKEN AS AN ACTION.

Run `flint shard start strk` if you haven't already.

# Workflow: Do With Operations

Use this workflow when the work of this session is tied to an Operations task. It runs beside the workflow of the launch mode: [[dev-wkfl-strk-strike_start]], [[dev-wkfl-strk-discuss_start]], [[wkfl-proj-create_and_do_task]], [[wkfl-proj-do_task]], a notepad, or no workflow. That workflow does the work. This workflow keeps the Operations task true. It reads the task before the work, it starts the task, and it posts few updates. Each update says where the work happened and where it belongs in Operations.

The work is tied to an Operations task when one of these is true:

- The Strike preamble carries an Operations clause: `Operations task id: <id>, org: <org>`. Strike then adds a sentence that names this workflow ([[dev-knw-strk-sessions]]).
- The Mesh task of the work has the frontmatter field `ops-task` (and `ops-org`).
- The operator gives an Operations task id or web URL for the work.

An agent can also find the Operations work of its Flint by **place**: the deliverables whose Flint place is this Flint (Stage 1). A task found only by place is a candidate. It is not a tie until the operator confirms it.

# Input

- The launch prompt, the Mesh task, or the operator message that names the Operations task
- The `ops` CLI on PATH with a signed-in credential (`ops whoami`)

# Rules

These rules apply the Safety Rules of the Operations shard (`init-ops`, when this Flint has it) to this workflow. Obey them in every stage.

1. **Write only to the linked task.** This workflow has three writes: `ops task start <id>`, `ops comment add <id>`, and `ops task done <id>`. Use them on the linked task only. Do not change its title, description, priority, owner, or place. Do not write to another task, a subsection, a deliverable, or a project. Do not change the Flint place of a deliverable.
2. **Read before you write.** Run `ops task context` before the first write. Take ids from the prompt, the Mesh task, the operator, or `--json` output. Never construct or guess an id.
3. **Post few comments.** Post one update at each meaningful point: start, blocker, review stop, and done. Do not post a comment for a progress step, a test run, or a commit. Do not post one point twice. Before you post, read the newest comments (`ops task context <id> --org <org> --comments 5`). When this session already posted that point and nothing changed, do not post again. A resumed or relaunched session does not post a second start update.
4. **Sign each comment.** The last line of each update is the agent signature line: `_An agent wrote this comment through the ops CLI (Orbh session <session id>)._` Take the id from `$ORBH_SESSION_ID`.
5. **Check before you repeat a write.** A network error (exit 4) can come after the service applied the write. Before you repeat a write, read the task context and look for your comment or the new status. When the write landed, do not repeat it. When you cannot tell, tell the operator. Never repeat `ops comment add` without this check.
6. **Respect the data mode.** When the Strike clause says `data: dev`, the id is a dev id. Start every `ops` command of the task with `OPS_DATA_MODE=dev ndv run`, the reads and the writes alike. Before the first read, run `OPS_DATA_MODE=dev ndv run ops mode --json` and check that `dataMode` is `dev`. When it is not, read and write nothing, and tell the operator. Do not use `ops` alone or `ndv run ops` alone for a dev id: the `ops` on PATH can be a prod build, and the dev launcher still follows an inherited `OPS_DATA_MODE` or the stored data mode. Never send a dev id to prod data, and never post an update through a deployment that the check did not confirm.
7. **Respect the derived state.** A task in a blocked subsection cannot go to `done` (`subsection_blocked`). Tell the operator. Do not force a status.
8. **Write in Simplified Technical English.** Each update uses ASD-STE100 style: short sentences, one idea per sentence, active voice. Commands, ids, and SHAs stay exact.

# Actions

## Stage 1: Find the Operations Task

Find the linked task in this order. Stop at the first source that names one.

1. **The Strike preamble.** The Operations clause of the task phrase gives the id and the org. [[dev-wkfl-strk-strike_start]] also records them as `strike-ops-task-id` and `strike-ops-org`.
2. **The Mesh task.** The frontmatter fields `ops-task` (the id) and `ops-org` (the org) of the Mesh task of the work.
3. **The operator.** An id or a web URL. `ops` accepts a URL in place of an id.
4. **The place of this Flint.** Use this source only when the other three name nothing and the operator asks you to report the work to Operations.

   ```bash
   flint org show --json                       # flint.id, flint.name, and address
   ops org list --json                         # the orgs of the credential
   ops deliverable list --org <org> --json     # once for each org
   ```

   A deliverable belongs to this Flint when its `flintAddress` is the `address` of this Flint, or its `flintId` is the `flint.id` of this Flint. A deployment before (Task) 317 has no `flintAddress`, and an older Strike tie can hold the Flint name in `flintId`. For each deliverable that belongs to this Flint, read `ops deliverable show <deliverableId> --json`. Find the subsections and the tasks that match the work. Show the candidates to the operator: the title, the place (`Project / Deliverable / Subsection`), and the web URL. Ask which task the work belongs to. Write nothing until the operator names one. When no deliverable belongs to this Flint, say so and stop this workflow.

Record the link on the session:

```bash
flint orbh session set strike-ops-task-id "<Operations task id>"
flint orbh session set strike-ops-org "<Operations org>"
```

When the work has a Mesh task, write the link into its frontmatter, so a later session finds it:

```yaml
ops-task: <Operations task id>
ops-org: <Operations org>
```

When the Mesh task does not exist yet (for example, strike_start scopes it later), add the two fields when it is created.

## Stage 2: Read the Context and the Place

Read the task:

```bash
ops task context <id> --org <org>     # omit --org when no org is known
```

Keep these facts for the work and for the updates: the status, the place (`Project / Deliverable / Subsection`), the URL, the description, and the comments. The description and the comments are the operator's own words about the work. Use them as context. The workflow of the launch mode carries a summary into the scoped task.

Read the place of this Flint:

```bash
flint org show --json                 # the Flint name and its address, for example @/flint/nuu-operations
```

When `ops task context` fails:

| Exit | Meaning | Action |
|------|---------|--------|
| 3 | No credential | Tell the operator to sign in (`ops login`). Continue the work with no Operations write. |
| 1 with `*_not_found`, or no task | The task is deleted, or it is in another org | Tell the operator. Stop this workflow. |
| 4 | Network | Run the read again one time. When it fails again, continue the work with no Operations write, and tell the operator. |

Then read the status:

- `todo`: Stage 3 starts the task.
- `doing`: the task is already started (a Strike launch starts a tied row). Stage 3 posts the start update only.
- `done`: do not reopen the task. Tell the operator that the task is done in Operations, and ask whether the work continues. Post nothing until the operator answers.

## Stage 3: Start

The work starts when the implementation starts: the Mesh task goes to `in-progress`, or the session starts the work with no Mesh task. Scoping is not the start. In [[dev-wkfl-strk-strike_start]], the start is the operator's **go**.

At the start:

```bash
ops task start <id> --org <org>       # only when the status is todo
ops comment add <id> --org <org> - <<'EOF'
<the start update>
EOF
```

Use the update format below.

## Stage 4: Work

Do the work with the workflow of the launch mode. Post no comment while you work. Post an update only at these two points:

- **Blocker.** The Mesh task goes to `blocked`, or the work stops on a decision that only the operator can make. Post the blocker update. Do not change the Operations status.
- **Review stop.** The Mesh task goes to `review`, after the WIP commits. Post the review update. Do not run `ops task done` at this point: the review comes first.

## Stage 5: Done

The Operations task is done when the work is reviewed and committed and the operator confirms. Ask the operator one question: "Mark the Operations task done?" Only after a yes:

```bash
ops task done <id> --org <org>
ops comment add <id> --org <org> - <<'EOF'
<the done update>
EOF
```

When the operator says no, or more work remains in Operations, post nothing more.

## The Update Format

An update is relative to its place. It names where the work happened (this Flint, its Mesh task, its repositories) and where it belongs in Operations (the place of the task). Write it in Markdown, through stdin (`-`), so the lines stay intact.

```markdown
**<Start|Blocked|Review|Done>** · <Flint name> (`<Flint address>`)

- Place: <Project> / <Deliverable> / <Subsection>
- Mesh task: (Task) NNN <Title>
- <Plan|Blocker|Changed|Result>: <one to three sentences>
- Commits: `<sha>` (<repository>), `<sha>` (<repository>)
- Verified: <the commands and their counts>
- Open: <the open items, or none>

_An agent wrote this comment through the ops CLI (Orbh session <session id>)._
```

| Point | Lines |
|-------|-------|
| Start | Place, Mesh task, Plan |
| Blocked | Place, Mesh task, Blocker (what blocks and the decision that is necessary), what is done so far |
| Review | Place, Mesh task, Changed, Commits, Verified, Open |
| Done | Place, Mesh task, Result, Commits, Open |

Drop a line that has no content (for example, no commits at the start). Name a commit by its short SHA and its repository. Name a Mesh task by its full title; Operations does not resolve a wikilink.

# Output

- The Operations task started, with at most one update for each meaningful point
- The link recorded on the session (`strike-ops-task-id`, `strike-ops-org`) and on the Mesh task (`ops-task`, `ops-org`)
- No write outside the linked task
