# Strike

Context and workflows for Strike, the local-first target-graph terminal app that launches Orbh sessions.

Strike lives at `apps/strike-tui` in the Main repository. It holds a personal target graph, ties selected parts of it to NUU Operations, and launches Orbh agent sessions against single rows. This shard is the context those launched sessions need.

## What This Shard Provides

| File | Purpose |
|------|---------|
| `dev-init-strk.md` | The four concepts, the target model, the mount boundary, the vocabulary |
| `knowledge/dev-knw-strk-strike.md` | Deep reference on Strike: kinds, modifiers, readiness, the Operations hierarchy, storage, house patterns |
| `knowledge/dev-knw-strk-sessions.md` | The Strike–Orbh tie: bindings, the launch chain, the composed prompt contract, what a session may assume |
| `workflows/dev-wkfl-strk-strike_start.md` | Read the launch preamble, load Strike context, then scope the target as a task |

## The Launch Prompt

A Strike-launched session wakes with a composed one-line prompt:

```
This session was launched from Strike for task '<title>' (id: <targetId>) under subsection '<subsectionTitle>' (id: <subsectionId>). Run `flint shard start strk`, then follow the wkfl-strk-strike_start workflow before other work. <operator text>
```

The subsection clause drops whole for a local-only row. The strike_start instruction is a per-launch toggle in the launch modal's prompt stage (`ctrl-s`), on by default.

## What This Shard Does Not Own

Strike declares no Mesh artifact type and creates no artifact folder. Task artifacts belong to the Projects shard — `strike_start` delegates to `wkfl-proj-scope_task` rather than duplicating the template.

## Structure

```
Shards/(Dev Local) Strike/
  shard.yaml                                  # Manifest (not prefixed)
  RELEASE.md                                  # Release changelog (not prefixed)
  dev-init-strk.md                            # Init file — shard context
  workflows/dev-wkfl-strk-strike_start.md     # Orient, then scope the target
  knowledge/dev-knw-strk-strike.md            # What Strike is
  knowledge/dev-knw-strk-sessions.md          # The Strike–Orbh tie
```
