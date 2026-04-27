---
title: "Engine Internals"
icon: "⚙️"
sources:
  - engine/
updated: 2026-04-25
created: 2026-04-27
---

# Engine Internals

This section is for people working on the engine itself — fixing bugs in `Sandbox.Engine`, adding native bindings, modifying the editor tooling, or just trying to understand how the runtime fits together. **If you're trying to make a game using s&box, you're in the wrong section** — go to [Build Games](../build-games/index.md) or the [Architectural Overview](../architecture.md) instead.

## What you came here for

Pick the page that matches what you're trying to do.

| You want to... | Go to |
|---|---|
| Get from "I cloned the repo" to "first PR merged" with structure | [Learning Path: Engine Contributor](learning-path-contributor.md) |
| Clone the repo and build it | [Building from Source](contributing.md) |
| Find which `Sandbox.X` project to modify for a given change | [Where to Make Changes](where-to-make-changes.md) |
| Understand the boot sequence | [Engine Launcher](launcher.md) and [Runtime Layout](runtime-layout/index.md) |
| Understand how hotload works under the hood | [Sandbox.Hotload](sandbox-hotload.md) |
| Understand how user code is compiled | [Sandbox.Compiling](sandbox-compiling.md) |
| Understand how the sandbox security boundary works | [Sandbox.Access](sandbox-access.md) |
| Understand reflection / `TypeLibrary` | [Sandbox.Reflection](sandbox-reflection.md) |
| Add or modify native (C++ → C#) bindings | `engine/Sandbox.Engine/Core/Interop/` and `engine/Tools/InteropGen` |
| Extend the editor (a custom Inspector, tool, or dock) | [Sandbox.Tools](sandbox-tools/index.md) and [Editor](../editor/index.md) |

## Repo layout

**`engine/`** — Engine source. Each `Sandbox.X` folder is one project.

| Path | Purpose |
|---|---|
| `engine/Sandbox-Engine.slnx` | Open this in Visual Studio / Rider. |
| `engine/Sandbox.Engine/` | The biggest project. Most of what you'll touch. |
| `engine/Sandbox.Hotload/` | Live-state migration during code reload. |
| `engine/Sandbox.Compiling/` | Roslyn-driven runtime compilation. |
| `engine/Sandbox.Reflection/` | TypeLibrary. |
| `engine/Sandbox.Bind/` | Property data-binding system (Razor UI). |
| `engine/Sandbox.Tools/` | Editor itself (Inspector, Mapping, Hammer integration). |
| `engine/Sandbox.Access/` | Sandbox security (assembly verification). |
| `engine/Sandbox.Razor/` | Razor UI compiler. |
| `engine/Mounting/` | External-game mount support (GoldSrc, NS2, Quake). |
| `engine/Launcher/` | Entry-point executables (sbox, sbox-dev, sbox-server, ...). |
| `engine/Tools/` | Build-pipeline tools (InteropGen, SboxBuild, ShaderCompiler). |

**`game/`** — Runtime files.

| Path | Purpose |
|---|---|
| `game/addons/base/` | The shipped base addon. |
| `game/addons/citizen/` | Default avatar. |
| `game/addons/menu/` | Main menu environment. |
| `game/addons/tools/` | Built-in editor tools. |
| `game/core/` | Engine baseline content. |
| `game/bin/` | Native binaries (Source 2 DLLs). |

For deep per-project documentation see the individual `Sandbox.X` pages on this section's TOC.

## How a single C# user line becomes a frame

Trace the path of a `Component`'s `OnUpdate` for context:

1. Native engine ticks (`engine/bin` Source 2 DLLs).
2. Native invokes `Sandbox.Engine.Core.EngineLoop` from the .NET host (`engine/Launcher/Shared/Startup.cs` set up the host on boot).
3. The engine loop walks the active `Scene`'s GameObject tree.
4. Each enabled `Component` has its `OnUpdate` invoked — the user's hot-loaded assembly's method body runs.
5. Position changes (`GameObject.WorldPosition = ...`) flow into `Sandbox.Engine.Systems.SceneSystem`.
6. From there into the native scene-object representation via the `Sandbox.Engine/Core/Interop/` interop layer.
7. Render gathers the new transforms next frame.

Each step lives in a different `Sandbox.X` project. Nothing magical; following the call stack across projects is how most contributors learn the codebase.

## Conventions to know

**Single-threaded by design.** The Source 2 scene graph and renderer require main-thread submission. Most engine code assumes single-threading. Use the `Task` system to offload work but always marshal back before touching scene state.

**No allocations in the tick path.** `Sandbox.Engine.Core.EngineLoop` runs every frame. Returning `new List<>` or formatting strings inline causes GC pauses that stutter the entire engine. Pool, cache, or use `Span<T>`.

**Reflection caches must be cleared on hotload.** If you cache `Type` or `MethodInfo` references, register a cleanup callback on the `[Event("hotload")]` event. Otherwise hotload silently breaks for users whose code touches your subsystem.

**`[Hide]` is a real signal.** Many components like `MeshComponent`, `MapObjectComponent`, `HammerMesh` are `[Hide]`-marked because they're added by tools, not manually. Don't add them to "drop directly" UI suggestions.

**Native bindings come from `InteropGen` or are hand-written.** The interop layer lives in `engine/Sandbox.Engine/Core/Interop/` (with the bootstrap shim in `Sandbox.NetCore`). If you're adding a new native function, you usually add it to a `.def` file under `engine/Definitions/` and let `engine/Tools/InteropGen` generate the wrapper. Hand-edit only when InteropGen can't express the marshalling you need.

## Pitfalls when modifying the engine

**Modifying `Sandbox.Engine` requires a full rebuild.** The Roslyn fast-path doesn't help here; you're changing the engine itself, not user code. `Bootstrap.bat` is the canonical full rebuild.

**`DllNotFoundException` after rebuilding.** Usually means your native (`game/bin`) is out of sync with the managed assembly's expected exports. Re-run `InteropGen` or pull the matching native binaries.

**`AccessViolationException` from native code.** A null or invalid pointer was passed across the C# → C++ boundary. Check the C# wrapper's input validation; the native side often assumes inputs are non-null.

**Tests pass locally but fail on the GitHub Actions PR check.** The CI runs against a clean checkout with a clean native build. If your local has stale `bin/` from a prior branch, you might be testing against the wrong native side. `Bootstrap.bat` cold to confirm.

## What lives in `engine-internals/` (this section)

Every `Sandbox.X` project has its own page on the TOC. The most-used ones:

- [Sandbox.Engine](sandbox-engine/index.md) — biggest project; Scene, Systems, Game, Editor, UI, Input, Resources
- [Sandbox.Hotload](sandbox-hotload.md) — heap-walking deep-swap upgrader
- [Sandbox.Compiling](sandbox-compiling.md) — Roslyn-based user-code compiler
- [Sandbox.Reflection](sandbox-reflection.md) — `TypeLibrary` and reflection helpers
- [Sandbox.Bind](sandbox-bind.md) — property data-binding system used by Razor UI; see also [Links](sandbox-bind-links/index.md) and [Proxy](sandbox-bind-proxy/index.md)
- [Sandbox.Access](sandbox-access.md) — assembly verification and sandbox boundary
- [Sandbox.Tools](sandbox-tools/index.md) — the editor (Qt-wrapped, ControlWidget, Inspector, MapEditor, etc.)
- [Sandbox.Razor](sandbox-razor.md) — `.razor` → C# compiler

Plus:

- [Mounting Internals](mounting.md) — VFS interceptors and external-game support
- [Engine Launcher](launcher.md) — boot sequence (Crash Reporter, sbox-dev, sbox-server, sbox-bench, sbox-profiler)
- [Runtime Layout](runtime-layout/index.md) — game directory structure and load order
- [Tools (Build Pipeline)](tools/index.md) — InteropGen, SboxBuild, ShaderCompiler, ShaderProc, CodeGen

If you want a higher-level conceptual view rather than per-project deep dives, [Engine Concepts](../engine-concepts/index.md) is the parallel section.

## Related Pages

- [Architectural Overview](../architecture.md) — the layered model both audiences share
- [Engine Concepts](../engine-concepts/index.md) — high-level subsystem overviews
- [Building from Source](contributing.md)
- [Where to Make Changes](where-to-make-changes.md)
