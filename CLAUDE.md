# GameSpice Physics

Physics as a choice a game makes, rather than as whatever the engine defaulted to: named collision layers in preset layouts, an engine profile that is applied and put back, and a table of surface materials. It does not simulate and it does not own bodies.

Read the family [CLAUDE.md](../../CLAUDE.md) first.

**The distributable is `addons/gs_physics/`.** It requires [GameSpice Core](../gs-core) and nothing else (`ln -s ../../gs-core/addons/gs_core addons/gs_core`, gitignored).

## Read before

| Read | Before |
| --- | --- |
| [docs/design.md](docs/design.md) | changing anything: named layers, putting settings back, the presets, the closed collision relation, `classify`, surfaces as numbers, what this does not do |
| [docs/threading.md](docs/threading.md) | adding a caller that casts: every space query must run inside the physics step; the audited list |
| [docs/history.md](docs/history.md) | the layout every game named and no body followed |

## Layout

| Path | What |
| --- | --- |
| `core/gs_physics_layer.gd` | `GSPhysicsLayer`: one named layer, `collides_with`, `query_only`, the assigned `bit` |
| `core/gs_physics_layout.gd` | `GSPhysicsLayout`: the set, `build()` closing the matrix, `layer_mask`/`mask_of`/`solid_mask`, four presets and `custom` |
| `core/gs_physics_profile.gd` | `GSPhysicsProfile`: every engine number (gravity, tick rate, ...) as a layered `GSConfig` |
| `core/gs_physics_surface.gd` | `GSPhysicsSurface`: what one material is like to collide with |
| `core/gs_physics_surface_set.gd` | `GSPhysicsSurfaceSet`: the table, `standard()`, how a collider maps to an entry (`tag`) |
| `runtime/gs_physics_world.gd` | `GSPhysicsWorld`: applies a profile and a layout, `classify`, puts settings back on exit |
| `runtime/gs_physics_query.gd` | `GSPhysicsQuery`: casts written against layer names (`shot_3d`) |
| `runtime/gs_tick_gate.gd` | `GSTickGate`: holds handlers heard between ticks until the server's next tick (docs/threading.md) |
| `examples/physics_selftest.tscn` | the suite: no world, no bodies, no renderer for the `core/` half |

Paths except `examples/` are in `addons/gs_physics/`.

## Running and testing

The family's two steps ([family CLAUDE.md](../../CLAUDE.md#validating-anything)), then `timeout 120 godot --headless --path . res://examples/physics_selftest.tscn`.

## Rules that bite

- Nothing outside a layout says a bit number; ask `layer_mask` / `mask_of`. [design](docs/design.md#a-collision-layer-has-a-name)
- Every `ProjectSettings` write has a restore; `core/` never touches the engine or a node. [design](docs/design.md#anything-that-writes-to-projectsettings-puts-it-back)
- Set `Engine.physics_ticks_per_second` as well as the project setting, or the tick rate waits for the next run. [design](docs/design.md#anything-that-writes-to-projectsettings-puts-it-back)
- Never name a method after an `Object` virtual (`_set`); hence `_write_setting`. [design](docs/design.md#anything-that-writes-to-projectsettings-puts-it-back)
- Declare `collides_with` one way; `build()` closes it and must keep refusing an unknown name. [design](docs/design.md#the-collision-relation-is-declared-once)
- Shots use `solid_mask()`, so `query_only` layers never stop them. [design](docs/design.md#where-the-presets-came-from)
- Naming layers changes nothing until a game calls `classify` on every body; never discard its `GSResult`. [design](docs/design.md#naming-a-layer-is-half-a-layout-the-other-half-is-classify)
- A game that moves bodies off bit 0 sets `GSFpsTunables.collision_mask` from the layout (it defaults to 1). [design](docs/design.md#naming-a-layer-is-half-a-layout-the-other-half-is-classify)
- Build the layout on every peer; only the profile is the server's. [design](docs/design.md#naming-a-layer-is-half-a-layout-the-other-half-is-classify)
- Surface fields (`liquid`, `break_health`, `hazard_damage`, `jump_scale`, `solid`) are numbers; the game decides what they do. [design](docs/design.md#surfaces-are-numbers-not-behaviour)
