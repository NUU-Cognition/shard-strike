---
description: "What Strike is — the two concepts, the subsection picker, the Operations page, the target model, kinds and modifiers, readiness, the Operations hierarchy, and storage"
orbh-sessions:
  - "[[5632c0de-bb30-49ff-975d-96d93e428d97]]"
  - "[[65228e07-2f8f-4f64-938a-5fbe4825c339]]"
  - "[[c174bf33-b199-4688-b89b-f26247582f78]]"
  - "[[f164a14a-214a-4da4-9332-9d77483bce25]]"
  - "[[6516416f-28e4-41e7-9364-3887f07b4eb4]]"
  - "[[00c93799-6730-4346-b597-22b2550b2cc1]]"
  - "[[6b9d1c77-c2bd-464f-a404-fbc9b59141c3]]"
  - "[[12e6ab26-ddf4-4e5f-9ce7-def93bbf84a3]]"
  - "[[8f4b956a-490e-4b34-a91b-0e3faf67ec5d]]"
  - "[[2177139a-5247-4a1a-914e-8e8bc53358e2]]"
  - "[[17071312-cf12-402b-9f74-64304657a95d]]"
  - "[[f2c7a286-a3e8-4f0a-a65a-a1e81adb823d]]"
  - "[[47411044-3fbf-4dcc-b869-9641652bcce7]]"
---

# Knowledge: Strike

Strike is a local-first terminal app for a personal target graph, tied selectively to NUU Operations. This file is the reference for its model. Read it when you must reason about a Strike row, or when you work on Strike itself.

## Where Strike Lives

| | |
|---|---|
| Source | `apps/strike-tui` in the Main repository (`flint resolve codebase Main`) |
| Package | `@nuucognition/strike-tui`, private, ESM |
| Runtime | Node.js 26.1 or newer |
| Stack | OpenTUI (`@opentui/core`, `@opentui/react`), React 19, Convex client, chrono-node |
| Launcher | `~/.nuucognition/ndv/bin/strike` links to the entry that ndv activated. `bin/strike-prod.js` loads `dist/index.js`. `bin/strike.js` makes a silent development bundle in `.strike-dev/` from current source and starts it. `--version` shows `[dev]` in development mode ((Task) 222, (Task) 229). In a development build, `ctrl-alt-r` or `:restart` rebuilds current source and relaunches ((Task) 238). Production has no such gesture. |

The state icons are Nerd Font MDI glyphs. The terminal needs a Nerd Font.

## The Two Concepts

| Concept | Meaning | Gesture |
|---------|---------|---------|
| **Subsection** | One Operations subsection, held locally. A Strike subsection is a 1:1 match to an Operations subsection: `createScope` refuses a second one on the same `(orgSlug, subsectionId)`, and the engine method `scopeForSubsection(orgSlug, subsectionId)` is the one lookup every caller shares. There is no local-only subsection and no target outside one. | `n` opens a new tab on one; `w s` adds one to this tab; `w m` manages the subsections themselves |
| **Project backlog** | One project's loose tasks, held locally as a scope of the same shape ((Task) 225). 1:1 with the project: `createScope` refuses a second one on the same `(orgSlug, projectId)`, and `scopeForBacklog(orgSlug, projectId)` is its lookup. Its anchor is the `ops-backlog` row, and its members are the project's backlog tasks. | The destination picker lists one `Project / Backlog` row per project; choosing it runs `pullProjectBacklog` |
| **Root** | The subsection that orients the current tab. A tab holds one or more subsections and one of them is the root. | `]` rotates the root through the tab's set; `w r` picks it; `W r` pulls a subsection and roots the current tab at it |

The subsection is the unit ((Task) 215). In code the entity keeps the name `Scope` and the field `scopes[]`; in prose and on every operator-facing surface it is a **subsection**. `Scope.grounding` is required, so `autoNamed` is always true and the name follows canon.

### The Scope Grounding

`Scope.grounding` is a discriminated union with two members ((Task) 225).
Every call site handles both, and none may assume `ref.subsectionId` exists.

| `kind` | `ref` | `context` | Anchor mount |
|--------|-------|-----------|--------------|
| `subsection` | `{ subsectionId, deliverableId?, orgSlug? }` | project and deliverable | `ops-subsection` |
| `project-backlog` | `{ projectId, orgSlug? }` | project only — a backlog has no deliverable | `ops-backlog` |

`groundingRefId(grounding)` is the one Operations id a grounding names, and
`groundingKey(orgSlug, kind, refId)` is the complete identity the engine keys
1:1 on — the kind is part of the key, so a subsection and a project backlog
can never collide.

`PushBinding` grew the same second form. A binding with no `kind` is the
subsection form, which is every binding written before (Task) 225; a backlog
binding carries `kind: "project-backlog"` and a `projectId`. Read it through
`isBacklogBinding`, `bindingSubsectionId`, `bindingRefId`, `sameBinding`,
`bindingFromGrounding` and `groundingFromBinding` in
`src/model/push-destination.ts` — never by reading the fields raw.

### Moving a Subtree Between Scopes

`W t` is **Move to Scope…**: it moves the selected target and its whole
subtree from its current scope into any other scope, subsection or project
backlog alike. `W b` is the same move with no picker, into the current
project's backlog. Source of truth: `src/model/scope-move.ts`, which is pure —
it returns a plan and writes nothing.

**The destination pulls itself ((Task) 239).** `W t` opens the Operations
DESTINATION picker under a `move` intent — the same catalog picker every other
Operations gesture opens — so the rows are every subsection and every project
backlog the credential can reach, not only the units this machine holds. A row
the store already holds says `· held`; a row without that note is PULLED when
the operator chooses it, through the same `pullSubsection` or
`pullProjectBacklog` call the picker's own pull routes make. `W b` pulls the
project's backlog the same way instead of refusing. A unit the catalog cannot
name — Operations unreachable, one org's subscription in error — is still
offered from the store, because a local move needs no network.

**The pull is a FETCH, not part of the move.** It lands in its own undo entry,
taken BEFORE the move's atomic group is opened, so `u` after the gesture
reverts the MOVE and leaves the pulled unit standing. A pull that FAILS reports
its own error and plans nothing, so the graph is untouched. A pull that
succeeds under a move that is then REFUSED leaves the pulled unit held: the
pull was a separate act, and the unit is legitimately in the store now.
(Task) 225's rule that a move destination must already be held is gone; every
refusal `planScopeMove` makes is unchanged.

**The closure.** A row travels when it HOLDS THE SOURCE MEMBERSHIP, the moved
row transitively requires it, and NO row outside the closure also requires it.
A shared row — something the moved subtree and a sibling both require — stays
where it is, because moving it would drag a dependency out of a unit that
still needs it. The descent stops at three kinds of row and none of them
travels: a scope ANCHOR, an ARCHIVED row, and a row with NO source membership.
The named root always travels whatever holds it.

A row with no source membership is left WHOLE — no membership write, no
`pushBinding` rewrite, no canon placement. The move REPLACES one membership
with another, and such a row has none to replace. A foreign membership is
never disturbed.

**The rule is NOT the (Task) 215 tree-delete rule, and the difference is
deliberate.** `deleteScope` seeds with rows THIS SCOPE ALONE holds and spares
any row that ANY row outside the shrinking set requires — an archived row and
a scope anchor included. The move seeds with rows that hold the source
membership, and it ignores two kinds of requirer the delete counts: a SCOPE
ANCHOR (an anchor edge is scope structure, not a dependency — the source
anchor's own edges are exactly what the move rewrites) and an ARCHIVED
requirer (it holds no live structure). The delete may ignore neither: it
REMOVES rows and cannot be replayed, so a spared row is the only protection an
archived requirer has. A move removes nothing and `u` reverts it whole, so it
only has to protect LIVE structure.

**What one move changes,** and nothing else: the source membership becomes the
destination membership on every closure row (a foreign membership is left
alone); EVERY source-anchor edge into the closure moves to the destination
anchor, not only the moved root's; every live source-scope parent edge from
outside the closure closes, so the moved row cannot unlock a row in the source
scope; edges inside the closure stay unchanged; and every UNTIED closure row's
`pushBinding` becomes the destination.

**The apply order is fixed** (`SCOPE_MOVE_APPLY_ORDER`): drop the source edges,
write the memberships, add the destination edges, rewrite the bindings.
`applyScopeMovePlan` is the ONE consumer of that order, so no caller can drift
from it. `LocalClient.update` drops a row's direct anchor edge as a side effect
of a membership leaving, and that side effect records no inverse — so the move
brackets it on both sides, for every closure row the source anchor holds. Any
other order leaves `u` with an edge it cannot restore, or one it tries to
remove twice.

**One undo entry, all or nothing.** The whole local move lands in one
`journal.atomicGroup`, so `u` puts every membership, every edge, and every
binding back. A step that fails replays every inverse collected so far and
pushes NO undo entry, so a half-applied move can never survive.

**Canon follows, it never leads.** Only after the local commit does canon run,
one row at a time, best-effort, with every failure to the sync failure sink
under phase `replica`, named by the DESTINATION unit through `syncSinkRefOf`
(`backlog:<projectId>` for a backlog). Source of truth: `reconcileScopeMove`
in `src/ops/scope-move-canon.ts`. Reconciliation runs for **TIED** rows only:
a tie is already a canon fact, and a tied row whose Operations record says it
lives elsewhere is simply wrong. It places through the **destination**
organisation's `ops.forOrg` handle, never a row's own. An untied member of a
moved subtree is **never** pushed — it stays local, carries the destination in
its `pushBinding`, and still appears in `V`. Strike's one hard rule, that canon
is untouched until an explicit `P`, is unchanged.

**A move never crosses an organisation.** The destination scope must name the
same `orgSlug` as the source scope, and no tied closure row may name a third
one. Operations refuses a cross-org dependency, and every Operations write for
a tie goes through `ops.forOrg(orgSlug)` — an Org A task sent to an Org B
scope would carry an Org B id through the Org A client and diverge local from
canon permanently. The rule is enforced twice: the destination picker lists
only destinations in the source scope's org — a filter over the CATALOG since
(Task) 239, which spans every organisation the credential belongs to — and
`planScopeMove` refuses a cross-org destination and a foreign-org tie before
any write. An ABSENT `orgSlug` means
the HOME org on every side of that comparison — the source grounding, the
destination grounding, and a row's `mountOrg` alike — through the one shared
rule `normalizeOrgSlug` ((Task) 225 review). A missing value is a concrete
organisation, never a wildcard.

**A move refuses** on a scope anchor (`ops-subsection` or `ops-backlog`): an
anchor IS a scope, not a member of one. It also refuses an archived row, a row
that is not a member of the source scope, a move to the scope it is in, a
destination in another organisation, and a destination anchor the moved row
ALREADY REQUIRES — that edge would close a cycle, and the refusal comes from
the planner. The planner refuses the move before the first write.

**A destination choice applies the move immediately ((Task) 248).** The `W t`
picker applies the move when the operator selects a destination. The `W b`
gesture applies the move after it pulls an unheld backlog. The Backlog panel
applies the move when the operator selects a row. These paths do not open a
second confirmation prompt.

**ROW ORDER IS PART OF THE PLAN, unless the field is DECLARED order-free
((Task) 239 final review).** An earlier version sorted EVERY array, on the
claim that row order never changes what a move does. That claim is FALSE.
`memberships[].scopes` is order-significant live behaviour: `normalizeScopes`
trims and deduplicates while PRESERVING order, `update` writes that order, the
FIRST membership selects the tab root a row opens under, and the first GROUNDED
membership selects a tied row's push destination. Two membership arrays that
differ only in order push the same row to two different subsections. So the
rule is now: **preserve order by default, and sort only the arrays proved to
carry none.** `SCOPE_MOVE_PLAN_SET_FIELDS` names exactly TWO — `closureIds` and
`sharedIds`. No apply step iterates these arrays. It named `memberships` and `rebinds` too, on the claim
that each entry is one absolute write per DISTINCT row and therefore commutes.
The closing review REFUTED that ((Task) 239): `applyScopeMovePlan` calls
`update` once per entry, every `update` calls `touch` against a STRICTLY
MONOTONIC clock, so the array order decides which row keeps the newer
`updatedAt` — and that stamp is live state, shown by the detail view and sent
by replication. Both are ORDERED now. `addedEdges`, `droppedEdges` and
`canonRows` keep their order too, because `addRequire` APPENDS (so `addedEdges`
is the destination's child order, and `droppedEdges` is the source's child
order after `u`) and the canon tail calls `placeTask` in array order. Every
NESTED array keeps its order unconditionally; the declaration reaches top-level
plan fields only, so a new array is conservative until somebody proves it. The
claims are checked, not written down:
a declared set that repeats an element is refused, and a test reorders each
array over a REAL engine and reads EVERYTHING the apply writes — the anchor
children, the scope arrays, `pushBinding`, and the `updatedAt` order. Reading
only the graph is what let the two refuted claims stand.

**A value that is not in the plan is captured beside it.** A backlog placement
also reads the local priority of the moved row. `ScopeMovePlan` has no priority
field. The move takes a `ScopeMoveApplyInputs` record when execution starts.
The record contains one priority for each canon row. The canon tail uses this
record. It does not read a later priority from the store.

**The move picker hides completed destinations by default, held or not
((Task) 239 review).** "Does the catalog name this destination?" is decided
against the FULL catalog, and the completion filter runs LAST, over the joined
list of catalog rows and held rows. Deciding it against a filtered list made a
HELD COMPLETED subsection read as un-named, so the offline held-scope fallback
re-added it and it escaped the filter. `ctrl-d` unfilters both kinds together.
A held row the catalog cannot name carries no completion state at all — the
catalog is the only place that state lives — so it always shows.

**Opening the move picker writes nothing.** The shared picker caches a legacy
scope's Operations context on open, which is a store write and an undo entry.
The `move` intent SKIPS that cache: the move never reads the prefill, and
views stay pure ((Task) 239 review).

**Priority into a backlog.** A backlog task carries an `OpsTaskPriority`; a
target carries a local `priority`. The two ladders are one ladder under two
names, so the map is total: `low`/`medium`/`high` are identical, `immediate`
is `urgent`, and a row that states no priority lands in `queue` — the
Operations tier that means the same absence. Source of truth:
`backlogPriorityFromLocal` in `src/ops/status.ts`.

### The Subsection Picker

Operations enters the graph through one door: the flat DESTINATION picker (`ops-picker` modal, source `src/ops/picker.ts`). It lists subsections and, since (Task) 225, one `Project / Backlog` row per project — a backlog is a destination of the same kind, so it sits in the same flat list and filters across the same path. Choosing a backlog row runs `pullProjectBacklog`. It rides three live org-wide Convex queries per organisation (`projects:list`, `deliverables:listByOrg`, `subsections:listByOrg`), joins them into `Project / Deliverable / Subsection` rows, and filters as you type across the whole path. `ctrl-d` shows completed subsections. Every subsection-bound gesture opens it: `n`, the tab menu `n`, and `:tab new` pull the choice and open a NEW tab rooted at it; `W r` pulls and roots the CURRENT tab; `W s` pulls the chosen subsection as a unit and offers it to the tab; `w s` adds one to the tab's set; `w r` picks the root; `a L s` requires it; `:pull --subsection` pulls it. `W t` opens it too ((Task) 239), under the `move` intent: the rows narrow to the source scope's organisation, the source itself is excluded, a held destination is marked `· held`, and the choice is pulled if the store does not hold it and then moved. `W m` (promote) opens the same modal at its deliverable stage. There is no project browser: the Catalog tab view was removed in (Task) 194.

**Dependencies pull with the unit ((Task) 233).** A subsection's `dependsOn` subsections are part of its meaning — an open dependency blocks the objective — so every pull route above also pulls each dependency the store does not already hold, as a full unit, recursively down the chain. A failed dependency pull never fails the gesture; the derived dependency row keeps standing in until a later pull lands it. A scope anchor row (a boss, a backlog anchor) never wears the foreign-scope dim in the tree; an OPEN anchor's title renders pure white and underlined (`strikeTuiTokens.tree.boss`), and only doneness recedes it.

**Startup.** With no tab, or with tabs whose subsections the store no longer holds, Strike renders an empty pane reading "Choose a subsection to begin" and opens the picker once (`EMPTY_TABS_TEXT`, `src/tui/views/empty-tabs.ts`). Signed out, or with no Operations client, the same pane reads "Sign in with , then choose a subsection" and opens nothing.

### The Subsection Manager

`w` is the Subsection Menu, and it manages the **active tab's** subsection set: `s` adds a subsection through the picker, `d` removes one, `r` sets the root, `enter` roots at the selection's subsection, `m` opens the Subsection Manager, `L` edits the selection's subsection dependencies ((Task) 233) — a cycle-guarded multi-select over the org's subsections, the same list the Operations page opens with `L`; checked rows lead the list, the rows carry no org segment because a dependency must stay inside the subsection's own organisation (a server rule in `subsections.setDependencies`), the save is a canon write, not undoable, and a newly added dependency pulls as a full unit at once. (Task) 215 merged the old "Add Existing" item into `w s`, because the picker already lists the subsections the store holds. Apart from the dependency write, the menu never touches the subsection entities.

`w m` opens the **Subsection Manager** (source `src/tui/views/scope-tray.ts`, rows from `src/tui/scope-tray-model.ts`): a bottom panel over every subsection in the store, in store order. A row reads `NN ◇ Name  grounding · N targets` with a mark for the active tab — the root marker, a dot when the tab shows it, blank otherwise. Members count over live and archived rows, as `:scopes` counts them.

| Key | Action |
|-----|--------|
| `j` / `k` | Select |
| `space` | Show or hide the subsection in the active tab |
| `enter` | Root the active tab at the subsection |
| `ctrl-shift-↑↓` | Move the subsection one step in store order |
| `d` | Delete, behind a confirm that states the forecast |
| `f` | Go to the subsection's anchor target |
| `/` | Filter by name, id, or grounding label |
| `u` / `ctrl-r` | Undo / redo — every entity write joins the journal |
| `F` | Fullscreen (the shared panel preference) |
| `esc` / `q` / `w` | Close |

There is no create, rename, ground, or unground key. (Task) 215 deleted `c`, `r`, `g`, and `G` with the local scope: a subsection is created only by pulling an Operations subsection, and its name follows canon.

The delete confirm is not a bare "delete?". `scopes.delete` either **dissolves** the anchor into its parents, which removes only the anchor and splices its children up, or runs a **tree delete**, which removes the anchor and every subtree row that no other subsection holds and that nothing outside still requires. A member this subsection alone held goes with it, because no target survives outside a subsection ((Task) 215); a member that keeps another subsection stays. The confirm names which one will happen, how many rows it takes, and how many members survive elsewhere. `forecastScopeDelete` mirrors the engine's rule and is pinned against it in `test/scope-tray.test.ts`.

The Subsection Manager and the doing peek share the panel slot. Opening one closes the other. See (Task) 205.

### Commented Out Features

The following features are present in the source as commented code. They are not live features. Do not delete their source blocks.

| Task | Feature | Gestures |
|------|---------|----------|
| (Task) 215 | **Aim** — a tab locked on one target | `t` (Open Selected Target as Tab), `g a` (Next Actionable Target), `A` (View Toggle / lens cycle), `:aim`, `:map`, `:tab <ref>` |
| (Task) 215 | **Target root** — a tab rooted at a target instead of a subsection | `w t` (Root at Selected Target), `:root <target>`, the view menu items `x`, `2`, `R` |
| (Task) 230 | **Linked Tasks and Ordered Tasks** — overlay groups | Link, unlink, order, unorder, direct creation, group nudge, and automatic together-launch linking |

`ViewRoot { kind: "ref" }` stays in the type with no production producer. The vocabulary table still maps the word "aim", because `src/model/vocabulary.ts` is untouched.

The `overlay` field also stays in the type. The store still loads, validates,
repairs, saves, and replicates this field. Stored overlay data adds no status,
readiness, session, order, or display behavior in this version. The next
version can restore the blocks that have a `(Task) 230` comment.

### The Operations Page

`v O` opens the Operations page (source `src/tui/ops-page/`, pane machinery shared with the Arena in `src/model/panes.ts` and `src/tui/views/panes.ts`): a full-screen pane grid that manages projects, deliverables, subsections, and tasks in the terminal, live over the Convex client. Writes on the page are canon writes: live, and not undoable. Every organisation the credential belongs to is merged; the org name leads a row only when more than one org is listed. Glyphs: project `U+F00D6`, deliverable `U+F03D7`, backlog `U+F15AB`; a subsection shows the objective glyph.

| View | Panes | Keys |
|------|-------|------|
| Projects | left: the searchable project list (`Master Backlog` rows on top, one per org); right top: the selected project's deliverables (derived status, sections and task progress, closer due); right bottom: its Backlog and Queue preview | `tab`/`h`/`l` move focus across panes · `j`/`k` in a pane · `enter` on a project focuses the deliverables pane, on a deliverable opens the Deliverable view · projects pane: `/` filter · `c` new · `r` rename · `e` description · `u` next update · `s` status cycle · `d` delete — deliverables pane: `c` new · `r` · `f` Flint tie · `D` due date · `d` — backlog pane: `c` quick add · `C` to Queue · `r` · `space` · `!` priority · `P` promote from Queue · `m` place into a subsection · `o` do date · `R` rarity · `a` assignee · `K`/`J` reorder · `t` group by · `d` · `p` pull into the graph |
| Deliverable | header `Project / Deliverable · status · sections d/t · tasks d/t`; left (wide): Sections — subsections in order, closer last, tasks under each; right: the project's Backlog and Queue | sections pane, subsection row: `enter`/`z` fold · `Z` fold all · `H` hide done · `n` new subsection before · `c`/`C` new task · `r` · `D` · `K`/`J` · `L` dependencies · `!` priority colour · `d` · `y` pull as a unit · `Y` pull and root — task row: `r` · `space` (blocked rows say why) · `K`/`J` · `m` move · `b` send to backlog · `M` promote · `o` · `R` · `a` · `d` · `p` pull · `enter` on a tied row focuses its target — backlog pane: as above |

`esc` closes a filter, then returns from the Deliverable view to Projects with the deliverable selected, then closes the page; `q` closes the page. Derived statuses (subsection: done, blocked, in-progress, ready; deliverable: completed, blocked, in-progress, open) come from `src/ops/dag-status.ts`, a port of the web rules. Server refusals (`closer_locked`, `subsection_blocked`, `dependency_order_invalid`, …) flash with the server's message.

In the graph, the `W` menu offers `W t` (Move to Scope…) and `W b` (the same move into the current project's backlog, with no picker). Both are LOCAL subtree moves since (Task) 225, not canon-only placements, and since (Task) 239 both PULL a destination the store does not hold yet; see "Moving a Subtree Between Scopes".

## The Target

One node type carries the whole graph. Source of truth: `src/model/types.ts`.

| Field | Meaning |
|-------|---------|
| `_id` | Stable local id. This is what a launched session addresses a row by. |
| `title` | Operator text. Never transform it. |
| `status` | `todo`, `doing`, or `done` |
| `requires[]` | Target ids this one waits on — the DAG edges |
| `scopes[]` | Subsection ids (the code calls the entity a Scope). After (Task) 215 every row is born in the root subsection, so this is never empty on a live row; the field stays optional in the type |
| `doDate`, `dueDate` | `YYYY-MM-DD` or `YYYY-MM-DD HH:mm`, local wall-clock |
| `priority` | `low`, `medium`, `high`, `immediate` |
| `rarity` | Impact tier 1–5, Common to Mythic; unset renders quiet |
| `readiness` | Derived, never stored: `blocked`, `open`, `closed` |

### Type Variations

Every local-only variation of a target falls in one of five families. None is pushed to Operations. The families answer different questions, so a row composes one variation from each family, and exclusivity holds only inside a family. Source of truth: the comment block above `kind` in `src/model/types.ts`.

| Family | Field | Question it answers | Exclusive | Renders |
|--------|-------|---------------------|-----------|---------|
| Kinds | `kind`: `combo`, `attempt`, `ranged`, `stage` | How does this row itself behave — status derivation, blocking by its requires, its treatment of its parent | One per row | The state slot (gamified) or a trailing mark (plain) |
| Modifiers | `full`, `sideStrike` | How does this row read to its parent — the counter, the parent's readiness | Compose | `sideStrike` trails the state glyph; `full` is spelled by the counter alone |
| Overlays | `overlay`: `linked`, `ordered` | Which rows on one sibling row form a group, and with what group behaviour | One per row | A bracket right of the state glyph |
| Successor marks | `followUpOf`, `progressionOf` | Where did this row come from at a finish | One per row | Trail the state glyph — `followUpOf` marks the anchor, `progressionOf` the successor |
| Labels | `skillCheck`; `label`: `minion`, `henchman` | What meaning did the operator stamp on the row | `skillCheck` composes with everything; one class label per row | `skillCheck` trails the state glyph; a class label wears its own icon in the state slot |

Two label families sit under one word. A **pure label** (`skillCheck`) is a boolean and a trailing glyph. It changes no readiness, no status, no counter. A **class label** (`minion`, `henchman`) implies combo: the label rides the combo switch, refuses to outlive it, and replaces the combo mark with its own icon. Class labels are labels over combo semantics, not pure marks.

A new variation joins the family that answers its question. A meaning is a label. A behaviour is a kind or a modifier. A relation between rows is an overlay or a successor mark.

### Kinds

Four task kinds are mutually exclusive by construction. Absent means an ordinary target. A kind is local-only and is never pushed to Operations.

| Kind | Behaviour |
|------|-----------|
| `combo` | A full split whose subtasks cover all of its work. Status derives from them. Only the Finishing Blow closes it. Switching it on needs at least one subtask. |
| `attempt` | A record of one try at its parent. Born done and kept done. Pins to the top of its sibling group, blocks nothing, counts as no work. |
| `ranged` | A Ranged Target. Its own requirements never block it — readiness is never `blocked`. Toward its parent it behaves like an ordinary subtask. |
| `stage` | One phase of its parent. An open stage never blocks the parent. Parent completion auto-completes open stages and remembers the prior status; a parent reopen restores them. |

### Modifiers

Two local-only modifiers compose with any kind. Neither is pushed.

| Modifier | Behaviour |
|----------|-----------|
| `full` | The operator declared this grouping row's split complete. The counter drops its `+` and reads as an exact fraction. Adding a subtask does not clear it — only the operator does. |
| `sideStrike` | A Side Strike does not block its parent. The parent stays workable while this row is open. Counters, combo satisfaction, and objective roll-ups still count it. |

### Overlays

**Release state.** (Task) 230 comments out Linked Tasks and Ordered Tasks until
the next version. The text below describes the stored model and the source that
the next version can restore. None of these rules changes live behavior in this
version.

An overlay is a local-only group of targets on one sibling row. It sits on top of the `requires` graph and is never pushed. A target holds at most one overlay, in `overlay: { kind, group, ... }`. Source of truth: `src/model/overlay.ts` and the overlay verbs in `src/client/local.ts` (`joinOverlay`, `leaveOverlay`, `nudgeOrdered`).

Rules that hold for every kind: every member has the same complete set of live direct parents; a live group has two or more members, and a group left with one member dissolves; a member cannot change its parent set until it leaves; an `attempt`, a `combo`, and an objective cannot join; members sort adjacent in the tree and wear a two-cell bracket right of the status glyph (`open`, `mid`, `close`, or `solo` when rendered apart from the group).

A group is one unit to the rows around it (Task 204). `ctrl-shift-↑↓` inside a group reorders the member (a linked cluster orders by `tieBreak`, an ordered one by rank); off the top or bottom of the group it moves the whole group as a block past the neighbour row, under the usual tie rule. "Above Cursor" and "Below Cursor" in the create menu act on the whole group: above, the new target requires every member and takes over their unlocks; below, every member requires the new target, which takes over the union of their requirements. Every member keeps one shared parent set. While such a prompt is open the tree frames the group with a dim `─` rule above its first row and below its last. A child every member requires renders once, under the block's last member, and does not open a task column.

| Kind | Bracket | Behaviour |
|------|---------|-----------|
| `linked` (Linked Tasks) | `─┐ ─┤ ─┘` | Status gestures mirror across the group. An agent session launched on one member binds to all members. A joining row adopts the anchor's status. Keys: `a s` Link Task, `a o` Unlink Task; `c`, then `l` creates a Linked Task on the cursor's sibling row. |
| `ordered` (Ordered Tasks) | `━┓ ━┫ ━┛` | Members carry a dense rank from 1. Each member is gated by the member before it, as an implicit requirement that readiness, blocked reasons, cycle checks, and `g a` all read. A join appends at the end. Rank survives status gestures. `ctrl-shift-↑↓` on a member moves it one step. Nothing mirrors and nothing auto-starts. Keys: `a n` Order Task, `a x` Unorder Task; `c`, then `o` creates an Ordered Task on the cursor's sibling row. |

### Successor Marks

Both marks are born at a finish through the finish menu or the actions menu, and both splice the new row into the anchor's place: the new row requires the anchor and takes over every row that required the anchor. One undo entry reverts the whole group.

| Mark | Born from | Renders on | Meaning |
|------|-----------|------------|---------|
| `followUpOf` | `a F`, or the finish menu's `Create a follow up…` step | The anchor (`followedUp`, stamped at the store boundary) | Unplanned rework: the anchor closed, and later more work turned out to exist. Prefill `<title> Continued`. |
| `progressionOf` | `a G`, or the finish menu step, on a `doing` row with a complete subtree | The successor | The same effort continues one level up, born `doing`, with the anchor's agent session copied. |

### Skill Check

A skill check is a verification row: a check on work that was finished before it, or on work the operator names in its title. It is a pure label — the glyph U+F029A (nf-md-gauge) trails the state glyph in both vocabulary modes, and the row starts, finishes, blocks, and counts like any row.

| Surface | Gesture | Result |
|---------|---------|--------|
| The finish menu | `space` on a doing row, then the `Create a skill check…` step | The title prompt prefills the one word `Verify `. Enter finishes the anchor and splices the check into its place, wearing `skillCheck` and **not** `followUpOf`; the anchor wears no follow up mark. Blank or Esc creates nothing, and the run still finishes the anchor. The step is a finish owner: it excludes the follow up and progression steps. A second `space` on the menu is the plain finish; `ctrl-space` is the plain finish outside the menu. |
| The actions menu | `a k` | Sets or clears the label on the selected row. One undo entry. |
| The create menu | `c` … `k`, then a placement | A new row born with the label. `a` (Above Cursor) is the common case: the prompt prefills `Verify ` and the check splices above the cursor row (requires it, takes over its unlocks). `d` (Below Cursor) is the reverse splice. |
| The skill check tray | `S` | A bottom panel — the doing peek's panel over a different list — of every skill check the active tab's view can see, ordered doing, open, blocked, done. A row wears the gauge in its session gutter when unbound, and a leading `Verify ` is hidden from its title. `j`/`k` select, `enter` goes to the row, `space` starts a todo check or opens the finish menu on a doing one, `ctrl-space` finishes, `shift-space` resets, `o` agent menu, `f` focus, `a` actions, `F` fullscreen, `esc`/`q`/`S` closes, `D` switches to the doing peek (and `S` inside the doing peek switches to the tray). |

### Class Labels

`label` holds `minion` or `henchman`, the two Arena tiers below boss. A labeled row is always a combo. Keys: `a M`, `a H`, and the create menu's `M` and `H`. See Task 151.

### Engine Memory

`memo` holds bookkeeping the engine writes on the operator's behalf. `memo.priorStatus` is the status a stage held before its parent auto-completed it. It is spent when the parent reopens, or when the operator makes a manual status gesture. Never set it directly.

## The Mount Boundary

A target is local until it is mounted. The mount is an ordinary relation from a target to an external Operations record, and it is the only place the word "task" enters the model.

| `mountKind` | Meaning |
|-------------|---------|
| `ops-task` | Tied to an Operations task; `mountId` is that task's id |
| `ops-subsection` | This row is the subsection objective; `mountId` is the subsection id |
| `ops-backlog` | This row is a project backlog's anchor ((Task) 225); `mountId` is the PROJECT id |

`MountKind` has exactly these three members, and every switch over it stays exhaustive. (Task) 215 deleted `flag`, the mount that anchored a local scope and grounded a row outside a subsection. `isScopeAnchorMount(row)` is the one predicate for "this row is a scope, not a member of one" — use it rather than comparing to `ops-subsection` by hand.

**The predicate is the whole boundary, not a convenience.** An anchor is a scope and never work, in BOTH kinds. Every guard reads `isScopeAnchorMount`: no anchor takes a status gesture (`LocalClient.update`, `assertStatusMutable`, the reactive `update` and `statusMutation`), no anchor takes a local kind, no anchor is mounted or unmounted by hand (its tie is structural — delete the row instead; the mount menu offers only that explanation), no anchor is promoted (`promoteTargetToSubsection` and the TUI both refuse it), an undone mount restores an anchor by writing its mount back and never by `mount`, and a successful agent launch never STARTS one (`shouldStartLaunchedTarget`). Only the operator-facing WORD branches on the kind — "subsection"/"objective" against "project backlog" — never the condition.

The boundary also covers what an anchor IS, not only what it refuses ((Task) 225 review):

| Boundary | Rule for BOTH anchor kinds |
|----------|----------------------------|
| Derived status | An anchor's stored status is the roll-up of its members (`normalizeDerivedSubsectionStatus`). Finishing the last member settles the anchor; reopening one reopens it. This is what `BACKLOG_ANCHOR_STATUS_MESSAGE` promises when it refuses a direct set. |
| Overlay | An anchor never joins Linked Tasks or Ordered Tasks — its status is derived, so it cannot mirror one. |
| State glyph | An anchor is a grouping row and owns its state slot: `glyphKind` answers `objective`/`objectiveDone` for a subsection anchor and `backlog`/`backlogDone` for a backlog anchor. `state.backlog` is U+F15AB, the same mark the Operations page gives a project backlog. |
| Session binding | A backlog anchor takes its own key space, `backlog:<projectId>` (`sessionBindingKey`), because its `mountId` is a PROJECT id. `resolveSessionKeys` reads that prefix and skips the row, as it skips `sub:`. |
| Delete confirm | An anchor is an Operations tie, so the confirm says the LOCAL record goes and Operations remains (`targetDeleteConfirmText`). |

**One organisation rule, one spelling.** A stored `orgSlug` that is absent or blank means the HOME org — never the literal empty string, and never a wildcard. `normalizeOrgSlug(stored, homeOrgSlug)` in `model/scopes.ts` is the only place that rule is written; `LocalClient.normalizedOrg`, `orgMatches`, `planScopeMove` and the move destination picker all delegate to it. The planner and the picker read the engine's own home org through `StrikeClient.homeOrg()`. Two failures this replaces: a legacy scope with no org was refused a move into an explicit home-org destination, and a tie with no `mountOrg` was ACCEPTED into another organisation's scope, which sent a home-org task id through that organisation's client.

`mountLabel` is a denormalised external title, preserved when the record disappears. `mountMissing` marks a dangling mount.

`mountOrg` names the Operations organisation that owns the mounted record. Every tie names its org explicitly; a row from before ties carried an org (absent) is read as the client's default org. The org rides the tie, not the app: one graph holds rows mounted in several organisations, and a local `requires` edge may run between rows of two organisations. Operations refuses cross-org dependencies, so such an edge is local only and is never pushed. A push binding and a subsection grounding carry the same `orgSlug`. Every Operations read or write for a tie goes through `ops.forOrg(orgSlug)` — a handle over the same connection that stamps that org.

Creation under a section view is local-first. Canon is untouched until an explicit push (`P`). A new row is born in the tab's root subsection. `pushBinding` records the destination captured when the row was born, and `V` lists what was never pushed. A row may also hold a foreign membership in another subsection (`e s`).

`a o` is **Open in Operations** ((Task) 247): it opens the selected row's Operations record in the browser. A tied task lands on its deliverable view with `?task=<taskId>`, which reveals, scrolls to, and briefly highlights the row; a backlog placement lands on the project page; an objective lands on its deliverable view; a backlog anchor lands on its project page. An untied row refuses and names `P`. The base URL follows the build mode (`http://localhost:3088` dev, `https://ops.nuucognition.com` prod); there is no setting for it. Source of truth: `src/ops/open-in-ops.ts` and `src/open-url.ts`.

## The Operations Hierarchy

Strike reads and writes NUU Operations through a Convex client. Source of truth: `src/ops/types.ts`.

```
Project → Deliverable → Subsection → Task
```

| Entity | Notable fields |
|--------|----------------|
| `OpsProject` | `kind` (`project` or `operation`), `status` (`active`, `paused`, `archived`) |
| `OpsDeliverable` | `projectId`, `flintId` — the tie from a deliverable to a Flint |
| `OpsSubsection` | `deliverableId`, `dueDate`, `order`, `dependsOn[]`, `isCloser` |
| `OpsTask` | `subsectionId`, `status`, `priority` (`queue`…`urgent`), `ownerUserId`, `rarity` |

The **subsection is the unit of work**. It is what a Flint binds to, and it is what a launched session is told it works under.

## Cursor Movement

One grammar with two axes and three modifier levels. Plain keys move one step. Shift keys make one structural jump on the same axis. Ctrl keys jump by a screen block. Arrows and `hjkl` spell the same cells. Source of truth: `src/tui/keybinds.ts` and the navigation cases of `performAction` in `src/tui/app.ts`.

| Axis | Plain | Shift | Ctrl |
|------|-------|-------|------|
| Vertical | `j`/`k`, `↓`/`↑`: one row | `J`/`K`, `shift-↓`/`↑`: next or previous sibling | `ctrl-d`/`u`, `ctrl-↓`/`↑`: half a page; `ctrl-f`/`b`, `PageDown`/`PageUp`: one page |
| Horizontal | `h`/`l`, `←`/`→`: fold or go to the parent; unfold or enter the child | `H`/`L`, `shift-←`/`→`: go to the parent; leave the subtree forward to the next row at the parent indent | `ctrl-←`/`→`, `<`/`>`: previous or next task column |
| Extremes | `gg`/`G`, `Home`/`End`: first or last row | — | — |

Rules that hold across the grid:

- While task columns render side by side, every vertical gesture stays inside the cursor's display column. Only the column keys cross a bracket. In the stacked-band fallback and in the flat list the walk covers the whole list.
- `ctrl-→` on a row with a tie lands on the shared root it points at. `ctrl-←` on a shared root lands on its nearest referrer. Otherwise the jump lands on the row nearest the current screen line.
- The Finishing Blow row is its combo's last child for `J`/`K`, `H`, and `h`. Its only verb stays the finish gesture.
- `h` and `H` both arm a one-shot return memory, so `l` on the parent returns to the child the cursor left.
- Reorder (nudge) lives on `ctrl-shift-↑`/`↓`. It is an edit, not a movement. (Task) 230 makes a stored overlay member behave as an ordinary row.
- Placement mode and select mode route the same movement table as normal mode.

## Select Mode

`s` enters select mode with the cursor row already marked, so the first range gesture has an anchor. The mode marks target rows only; the Finishing Blow takes the cursor but never marks. Source of truth: `src/tui/selection.ts`.

| Key | Action |
|-----|--------|
| `space` | Mark or unmark the cursor row, and anchor the range on it |
| `shift-space` | Mark the band from the anchor to the cursor |
| `J` / `K` | Move one row and extend the band in one gesture |
| `t` | Mark the cursor row plus its visible subtree; a subtree already marked whole lifts out |
| `A` | Mark every target row the view shows |
| `i` | Invert the marks over the visible target rows |
| `c` | Drop every mark and stay in the mode |
| `a` | Open the bulk menu |
| `esc` | Drop the marks; a second press leaves the mode |
| `s` | Leave the mode |

Shift-click extends the band to the clicked row. A plain click still only points.

The band is recomputed from the anchor on every extension, so walking the cursor back toward the anchor shrinks it again. Marks made before the anchor was set stand through the whole gesture.

Bulk ops read the marks in row order, not in the order they were made, because an attach makes the op order the sibling order. Marks whose target the store no longer holds drop out, and the result reports how many.

## Vocabulary

One word-level transform runs over static user-facing strings at their render funnels. Source of truth: `src/model/vocabulary.ts`.

| Canonical | Gamification on | Gamification off |
|-----------|-----------------|------------------|
| target / targets | target / targets | task / tasks |
| combo / combos | combo / combos | folder / folders |
| subsection / subsections | boss / bosses | subsection / subsections |
| aim / aims | aim / aims | goal / goals |
| Finishing Blow | Finishing Blow | Close Folder |

The transform never runs over strings that embed operator titles. When an operator says "boss", they mean a subsection. When they say "folder", they mean a combo.

## Storage

All Strike state is machine-local. Do not edit these by hand.

**The stores split by DATA mode ((Task) 250, (Task) 253).** The table below names the prod paths. On dev data every file in it inserts `.dev` before `.json` (`strike-local.dev.json`, `strike-aims.dev.json`, …) and the runtime directory is `~/.nuucognition/strike-dev`, so a dev binary on dev data and a prod binary never touch the same file and both can run at once. `storeFilePath` and `runtimeDirPath` in `src/store-paths.ts` are the one place the rule is written; every `STRIKE_*` env override still beats the default in both modes. The pre-split data of this machine moved to the dev names, because it all came from dev use; a prod build starts fresh.

### The Data Mode

Strike carries two modes, and they answer different questions.

| Mode | Question | Source |
|------|----------|--------|
| **Build mode** | Which CODE runs? | Baked at build time ((Task) 222); read only by `getBuildMode()` |
| **Data mode** | Which environment's DATA does that code open? | `dataMode()` in `src/store-paths.ts` |

A prod build is always on prod data, so a prod binary can never be talked into reading dev files. A dev build defaults to dev data. One dev-only setting moves it to prod data, so a developer can run current source against the graph the operator actually works in ((Task) 253).

Four things follow the data mode together, and they cannot be separated. The store files and the settings file take the plain names, so the operator's launch profiles come along with the graph. The runtime directory becomes `~/.nuucognition/strike`, so a dev build on prod data and a prod build share ONE instance lock and cannot both run — one store has one writer. `DEFAULT_STRIKE_CONVEX_URL` becomes the prod deployment. And `strikeAuthEnv()` calls `resolveEnv("strike", { mode: dataMode() })`, so the credential is `auth.json`: each Convex deployment trusts one Clerk issuer, and a dev JWT sent to the prod deployment is refused with `NoAuthProvider`. `operationsBaseUrl(strikeAuthEnv().mode)` rides along, so `a o` opens the Operations web app that owns the record.

| Surface | Where |
|---------|-------|
| The setting | The **Data** row of the dev-only Diagnostics section on the settings page (`,`) |
| The stored value | `~/.nuucognition/strike-dev-data.json`, one key — EXEMPT from the data mode, because every other file moves when the mode moves and the flag would switch itself off |
| A one-shot override | `STRIKE_DATA_MODE=prod` or `dev`, above the file |
| The mark | `--version` prints `[dev · prod data]`, and the menu bar wears the `PROD DATA` badge beside `MOCK DATA` |

Two rules keep (Task) 250's protection. The **store version guard** refuses prod data while this binary's `LOCAL_STORE_VERSION` is newer than the prod store's `version`: opening it would migrate the operator's real store forward and the prod binary would then refuse it. The refusal falls back to dev data and names both versions. A **test run** stays on dev data: the suites run from source, so the build mode is dev and the stored flag would otherwise point every module-level path constant at the real store. `NODE_TEST_CONTEXT` is the signal; an explicit `STRIKE_DATA_MODE` still wins.

The change takes effect on the NEXT start. The path constants resolve once at module load, and a live instance holds the whole graph in memory and rewrites its store on every mutation, so swapping the store under a running instance is a different and much larger change. In a dev build `ctrl-alt-r` is the restart.

`LOCAL_STORE_VERSION` is **20** (`src/client/local.ts`). A newer reader migrates an older store forward once, atomically. An older binary refuses a migrated store and writes nothing, because each bump adds local truth the older engine cannot keep. Before any migrating write the engine copies the store to `<path>.pre-v20.bak`. v19 dropped what the subsection-only model could not hold: every ungrounded scope by the tree-delete rule, every target in no subsection, and any duplicate scope on one subsection. v20 adds the project backlog as a scope ((Task) 225): a scope may name a project backlog, a row may carry the `ops-backlog` mount, and a `pushBinding` may name a project instead of a subsection. A v19 engine reads none of the three — it refuses a backlog grounding outright, and it would strip an `ops-backlog` mount and a backlog binding from every row it rewrote. The engine reports the counts once, on the first start after the migration. There is no backward-compatibility code for the old model.

The tabs file is at version **8** (`src/tui/tab-store.ts`). It gets no backup — it is a view, and the graph is in the store. A tab whose subsections the store no longer holds is dropped on load, and an entry the new model cannot read is dropped, never mapped. When no tab survives, the startup path above runs.

| Path | Holds |
|------|-------|
| `~/.nuucognition/strike-local.json` | The graph and tabs' store — one flat file for every organisation |
| `~/.nuucognition/strike-aims.json` | Tabs. The name is historical; the file holds tabs, not aims. `STRIKE_AIMS_PATH` points at it |
| `~/.nuucognition/strike-orbh.json`, `strike-people.json`, `strike-fights.json` | Sidecars: session bindings, the people cache, fights |
| `~/.nuucognition/strike.json` | The credential and the client's default `orgSlug`, mode `0600` |
| `~/.nuucognition/strike-settings.json` | UI settings — never graph state. People is always on. Agent Sessions defaults on. |

There is one store and no home organisation. Ties inside the store name their own organisation (see `mountOrg` above); there is no org switch. The `orgSlug` in `strike.json` is only the default org for a raw id given without `--org` (the personal workspace after login). `:org` lists the organisations the credential belongs to.

The subsection picker (`a L s`, `W s`, `:pull --subsection`) lists every organisation the credential belongs to. When more than one org is listed, each row leads with the org name: `Org / Project / Deliverable / Subsection`. No other surface shows the organisation.

The per-organisation stores of the earlier layout (`strike-local.<org>.json` and their tabs and sidecars) are unified once by `apps/strike-tui/scripts/unify-strike-stores.mjs`, which remaps local ids, stamps every tie with its org, and backs the originals up.

### Replication

A subsection is the unit, so a subsection is the thing that replicates ((Task) 224). Every client that holds a subsection pushes its rows into the Convex table `strike_replica_targets` and subscribes to that subsection's rows. Two clients signed into the same org and holding the same subsection converge, including rows that were never pushed as Operations tasks. There is no sync gesture and no merge prompt. Source of truth: `src/ops/replica.ts` (write), `src/ops/replica-apply.ts` (read), `convex/strikeReplicas.ts` in `apps/nuu-ops` (server).

**What travels.** The local truth of every row in the unit: `title`, `body`, `status`, `requires`, `scopes`, `doDate`, `dueDate`, `priority`, `rarity`, `kind`, `full`, `sideStrike`, `overlay`, `followUpOf`, `progressionOf`, `skillCheck`, `label`, `memo`, `tieBreak`, `archivedAt`, `completedAt`, the mount identity (`mountKind`, `mountId`, `mountOrg`, `mountLabel`), `pushBinding`, `createdAt`, `updatedAt`. Derived and presentation fields never travel: `readiness`, `tiePending`, `ownerUserId` and `mountMissing` are re-derived by each client. The local `_id` never travels.

**Identity.** A local `_id` is machine-minted, so every replicated row also carries a `replicaId` — a UUID v4, minted at first replication and persisted on the row. The persisted field IS the id map. `requires`, `followUpOf` and `progressionOf` travel as `replicaId`s; `scopes` travels as Operations subsection ids. An edge whose far end holds no shared identity stays local and is dropped from the payload; nothing is invented on the wire.

**The `backlog:` rule.** A project backlog replicates as a unit of its own under the derived identity `backlog:<projectId>` — the twin of the `sub:` rule below, and for the same reason ((Task) 225). Its anchor identity is never minted, an arriving `backlog:` row maps onto this machine's anchor for that project, and a backlog tombstone never deletes the local anchor.

**The `sub:` rule.** A unit's objective row IS the subsection, so its identity is DERIVED — `sub:<subsectionId>` — never minted and never written into `Target.replicaId`. Each machine minted its own anchor when it pulled the subsection, so two minted ids would put two anchors in one unit; both peers compose the same derived string instead. An arriving `sub:` row maps onto THIS machine's anchor for that subsection: it never creates a row, and an objective tombstone never deletes the local anchor. An objective's `requires` list is the unit's root count, which the server caps at 200; when the objective row is therefore absent from a snapshot while live members are present, the anchor's edge set is DERIVED from the arriving member graph's roots instead, so the cap stops mattering on apply.

**Last writer wins.** The server stamps `rev` and `serverUpdatedAt`, so no client clock enters the rule. An arrival applies when its `rev` is newer than the newest rev this client already holds for that identity. A rev this device produced counts as held, so a client never applies its own echo; and an applied arrival is absorbed into the upload baseline, so it is never pushed back. A locally edited row whose push is still owed keeps the local version — the owed push supersedes it on the server. Each apply pass therefore pushes FIRST and reads arrivals second.

**Tombstones.** A delete travels as a `deletedAt` flag and the server keeps the document, so a client that was offline during the delete still learns of it. A tombstone from unit S removes only the row's S membership; the row itself is deleted only when it loses its last local membership. The apply then sweeps its edges and repairs the overlay group, exactly as a local delete does. A local delete is never inferred from absence: the store persists a pending-tombstone journal (`replicaTombstones`, written when a replicated row is deleted), the intent pushes best-effort, clears on server ack, and re-pushes on start. A live server row absent locally with no pending intent always materializes — peer data always beats inference.

**Canon still wins on a tie.** Operations owns a tied row's title, status, dates, order and creation moment. A tied task also gives up `body`, `priority` and `rarity`; a tied SUBSECTION keeps them, which is what makes an objective row worth replicating. Replication writes only the local truth Operations never learns. An arriving row with an ops-task mount re-ties by `mountId` through the existing pull machinery rather than creating a second row for the same task; when two clients each pulled the subsection and each minted an id for it, the lexicographically smaller id wins and the loser is tombstoned by the next diff.

**The sink.** Replication is best-effort behind the local commit. Every failure — a dropped socket, a typed refusal, a subsection above the server's 2000-row subscription cap (`strike_replica_subsection_too_large`) — reaches the sync failure sink under phase `replica`. Nothing rolls back a local edit, and a refused read leaves the unit exactly as local as it was; it never shows an empty unit.

**Cross-org.** Replication is per org. A membership or an edge into another organisation stays local, exactly as a cross-org tie does.

## Development restart

A development Strike can rebuild current source and relaunch without a quit ((Task) 238). Production has neither gesture.

| Gesture | Result |
|---------|--------|
| `ctrl-alt-r` | Works from every surface, the same way `ctrl-c` and `ctrl-t` do |
| `:restart` | The command-line form |

The running TUI cannot reload its own bundle in place. It shuts down, then exits with code `75`. The development launcher sees that code, rebuilds `.strike-dev/`, and starts a new app process.

## House Patterns

When you change Strike itself, these hold:

- No polling. Process exit, filesystem change, and subscription events are the signals.
- Views stay pure. Rendering never writes.
- No store writes inside subscription callbacks.
- Teardown stays untouched unless the task is about teardown.
- Verify with `pnpm typecheck`, `pnpm lint`, every `test:*` script, and `pnpm build` with `dist/` rebuilt.
