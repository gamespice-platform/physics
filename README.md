[![Physics - GameSpice addon](docs/banner.png)](https://moddingcommunity.com/a/55-gs-physics)

Named collision layers and named physics settings for Godot, so you stop writing `1 << 3 | 1 << 7` and stop hunting through `ProjectSettings` for solver numbers.

- **Layouts** give each collision layer a name and declare who collides with whom once.
- **Profiles** are physics settings (tick rate, gravity, solver iterations) named for how they feel. A dedicated server can switch profile from the command line.
- **Surfaces** map a collider to a material (friction, restitution, impact sound), so a ray hit can tell you it hit concrete.

The addon only decides what Godot is told. Godot still does the simulation.

## Install

1. Copy `addons/gs_physics/` and `addons/gs_core/` into your project.
2. Enable both in *Project > Project Settings > Plugins*.

Needs Godot 4.7 or newer.

## Use

```gdscript
var physics := GSPhysicsWorld.new()
physics.profile  = GSPhysicsProfile.arcade_shooter()
physics.layout   = GSPhysicsLayout.shooter_3d()
physics.surfaces = GSPhysicsSurfaceSet.standard()
add_child(physics)

var res := physics.setup()
if not res.ok:
    push_error(res.error.message)

physics.classify(player_body, &"player")   # layer and mask, by name
var hit := physics.query().shot_3d(world, muzzle, muzzle + dir * 100.0)
if not hit.is_empty():
    var surface: GSPhysicsSurface = hit["surface"]
    audio.play(surface.impact_sound)
```

`GSPhysicsWorld` remembers every setting it changed and restores them when it leaves the tree. A server that switches between two games therefore never keeps the first game's gravity.

## Layouts

| Layout | What it is |
| --- | --- |
| `shooter_3d` | Named layers for a first-person shooter, including player and NPC clip volumes, debris that touches only the world, and a query-only hitbox layer for lag-compensated hit registration. |
| `sandbox_3d` | The shooter layout plus `held_prop` and `frozen_prop`. A held prop does not push the player holding it. |
| `platformer_2d` | Side-on. One-way platforms have their own layer. |
| `top_down_2d` | No gravity, many bodies, players pass through each other. |
| `GSPhysicsLayout.custom()` | Your own layer list. |

Each layout declares a collision relation once ("debris collides with world") and `build()` makes it symmetric. An unknown layer name is an error at build time, not silently ignored.

## Profiles

| Profile | Feel |
| --- | --- |
| `arcade_shooter` | 128 Hz, gravity 20, no interpolation, characters are not pushed by bodies. |
| `grounded` | 64 Hz, gravity 12, 16 solver iterations, interpolation on. |
| `sandbox` | 24 solver iterations so crate stacks hold, aggressive sleeping so many settled props are cheap. |
| `floaty_platformer` | Long jumps, generous slopes. |
| `top_down` | No gravity. |

These numbers are the defaults of each profile, not limits. Every profile is a `GSConfig`, so values can be overridden in layers: defaults, then a JSON file, then environment variables, then the command line, for example `--physics-tick-rate 128`.

## Surfaces

`GSPhysicsSurfaceSet.standard()` has 11 materials with friction, restitution, density, sheet thickness, movement multipliers and sound/effect ids. `for_collider()` looks a surface up from node metadata, then groups, then the node name (a best guess, for mapper-built levels).

This is not GameSpice Player Controller's surface table. That one scales player movement; this one tells the solver about rigid bodies.

## More

- [docs/design.md](docs/design.md) for the rules behind the design
- [docs/threading.md](docs/threading.md) for threading notes
- [docs/history.md](docs/history.md) for past bugs

## Used by

These games vendor it:

- [Hungario](https://zeesgames.com/hungario)
- [Smash Copter](https://zeesgames.com/smash-copter)
- [Prop Hunt](https://zeesgames.com/prop-hunt)
- [Hide n Seek](https://zeesgames.com/hide-n-seek)
- [Airdrift](https://zeesgames.com/airdrift)
- [Deathrun](https://zeesgames.com/deathrun)
- [Zombie Zone](https://zeesgames.com/zombie-zone)
- [Fatal Contact](https://zeesgames.com/fatal-contact)
- [Playlot](https://zeesgames.com/playlot)
- [Murder Mystery](https://zeesgames.com/murder-mystery)
- [Wipeout](https://zeesgames.com/wipeout)

The whole collection is at [zeesgames.com](https://zeesgames.com).

## Status and licence

Part of the [**GameSpice**](https://gamespice.net) platform: a family of Godot addons, MIT licensed and built with Claude Code. The headless suite is `res://examples/physics_selftest.tscn`. The addon is used in play: the games in [Zee's Games](https://zeesgames.com) are built on GameSpice addons and run on live servers. If you hit a problem, please [open an issue](https://github.com/gamespice-platform/physics/issues).

MIT licence, see [LICENSE](LICENSE). The only closed part of the [**GameSpice**](https://gamespice.net) platform is the web back end; you can use your own instead of GameSpice's.
