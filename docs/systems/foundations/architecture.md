---
title: Foundational Systems Architecture
icon: "⚙️"
sources:
  - engine/Sandbox.System
updated: 2026-04-25
created: 2026-04-27
---

# Foundational Systems Architecture

If you've ever opened a `Component`, written `[Property] public float Speed { get; set; }`, called `Log.Info(...)`, multiplied something by `Time.Delta`, or used `MathX.Lerp` — you've already used `Sandbox.System`. It's the layer that has no engine in it.

This is the first project in the dependency chain. Everything else (`Sandbox.Engine`, `Sandbox.Tools`, `Sandbox.Hotload`, etc.) references it. It deliberately knows nothing about scenes, GameObjects, networking, or rendering — its job is to provide the language-level building blocks the rest of the engine assumes.

## What lives here

The sub-folders of `engine/Sandbox.System/`:

| Path | What it holds |
|---|---|
| `Math/` | `Vector3`, `Rotation`, `Transform`, `BBox`, `Curve`, `Gradient`, `MathX`, plus `Interpolation/` helpers. |
| `Time/` | `TimeSince`, `RealTime`, `Application` clock state. |
| `Logging/` | `Log`, `Logger`, NLog integration. |
| `ConVar/` | The console-variable system. |
| `Collections/` | Pooling, ring buffers, custom collections. |
| `Attributes/` | `[Property]`, `[Range]`, `[Group]`, `[ToggleGroup]`, `[ActionGraphNode]`, etc. |
| `Localization/` | String localization helpers. |
| `Graphics/` | `Color`, texture-format constants. |
| `Html/` | Minimal HTML parsing for label rendering. |
| `UI/` | Style and layout primitives consumed by the UI system. |
| `CodeGen/` | Source generators that ship with the API. |
| `SerializedObject/` | Data-driven property-set machinery. |
| `Utility/` | `Tracer`, `BytePack`, hashing, generic helpers. |

A typical game-dev's mental map of `Sandbox.System` is just "the math, the attributes, and `Log`" — and that's enough. Most code never reaches into the corners.

## Why it's separated

Three reasons it's its own assembly:

1. **Fast iteration.** When you're writing math helpers, you don't want to rebuild the whole engine. `Sandbox.System` builds in seconds.
2. **No transitive native dependencies.** It compiles cleanly without any C++ interop. Tools, generators, and tests can reference it freely without dragging in `engine2.dll`.
3. **Source-generator availability.** The generators in `Sandbox.System/CodeGen/` need to be loadable by Roslyn during compilation, before any of `Sandbox.Engine` is built. Putting them here makes that ordering work.

## Things that look like engine APIs but live here

Some surfaces feel like part of the engine but are actually defined in `Sandbox.System`:

- `[Property]`, `[Range]`, `[Group]`, `[ToggleGroup]`, `[ShowIf]`, `[HideIf]` — `Sandbox.System/Attributes/`
- `Log.Info`, `Log.Warning`, `Log.Error`, `Logger` — `Sandbox.System/Logging/`
- `Vector3`, `Vector2`, `Rotation`, `Transform`, `BBox`, `Plane`, `Ray`, `MathX`, `Curve`, `Gradient`, all interpolation helpers — `Sandbox.System/Math/`
- `Time.Delta`, `Time.Now`, `TimeSince`, `RealTime`, `Application.AppId` — `Sandbox.System/Time/`
- `ConVar` and the `[ConVar]` attribute — `Sandbox.System/ConVar/`

The reason this matters: when you're trying to navigate the engine source from a symbol you use every day, you'll find these in `Sandbox.System/`, not in `Sandbox.Engine/`.

## What's *not* here

`GameObject`, `Component`, `Scene`, `Rigidbody`, `MapInstance`, `Sound`, `Material`, `Model`, `Texture`, `Camera`, networking attributes (`[Sync]`, `[Rpc.*]`) — all of those live in `Sandbox.Engine`. As soon as something touches scenes, the asset system, the GPU, or the network, it leaves `Sandbox.System` behind.

## Related

- [Engine Internals](../../engine-internals/index.md) — the per-project reference for `Sandbox.System` and everything that depends on it.
- [Architectural Overview](../../architecture.md) — where `Sandbox.System` sits in the layer diagram.
