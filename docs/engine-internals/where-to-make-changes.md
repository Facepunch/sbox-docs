---
title: "Where to Make Changes"
icon: "🧭"
created: 2026-04-25
updated: 2026-04-25
sources:
  - engine/
---

# Where to Make Changes

## A concrete example: adding `[Range(0, 100)]`

Suppose you want to add a new property attribute — `[Range(min, max)]` — that clamps a numeric `[Property]` field in the Inspector. Where do you go?

You walk it through three projects:

1. **`Sandbox.System/Attributes/`** — define the attribute class itself: `RangeAttribute : Attribute` with `Min` and `Max` properties. This is where every other property attribute (`[Property]`, `[ShowIf]`, `[Title]`) already lives, so your new one belongs alongside them.
2. **`Sandbox.Reflection`** — if the attribute needs runtime semantics (clamping when the value is set programmatically, not just via the editor), wire it into the `TypeLibrary` walk so it's discoverable for any field.
3. **`Sandbox.Tools/ControlWidget/`** — add or modify the editor UI that draws the property. The `FloatControlWidget` reads `RangeAttribute` off the field and renders a slider instead of a textbox.

Then `Sandbox.Test` for tests, and you're done. One change, three projects, in a predictable order: *attribute definition → runtime behaviour → editor UI*.

That's the pattern the rest of this page generalises. Most contributions touch one project plus its tests; some, like the attribute example, are cross-cutting and the table below tells you which projects participate.

## The general rule

You want to fix a bug or add a feature. Which project do you touch? This page maps common contribution targets to the engine project that owns them.

If your topic isn't on this page, the rule of thumb is: search the engine for the user-facing API name, see which project the file lives under, work outwards from there. Most contributions touch one project plus its tests.

## Game-facing API changes

When you're changing what game devs see (adding a `[Property]`, adjusting a `Component`'s lifecycle, exposing a new method on a system).

| Topic | Project | Typical files |
|---|---|---|
| Component base class, lifecycle hooks | `Sandbox.Engine` | `Scene/Components/Component.cs`, `Component.Update.cs`, `Component.Loading.cs` |
| `GameObject` API | `Sandbox.Engine` | `Scene/GameObject/GameObject.*.cs` |
| Scene class | `Sandbox.Engine` | `Scene/Scene/Scene.*.cs` |
| `Scene.Trace` | `Sandbox.Engine` | `Scene/Scene/Scene.Trace.cs` |
| Built-in components (`Rigidbody`, `Collider`, `ModelRenderer`, `CharacterController`, `PlayerController`, etc.) | `Sandbox.Engine` | `Scene/Components/<group>/<Class>.cs` |
| Math types (`Vector3`, `Rotation`, `Transform`, `BBox`, `Angles`) | `Sandbox.System` | `Math/` |
| Property attributes (`[Property]`, `[Range]`, `[ShowIf]`, etc.) | `Sandbox.System` | `Attributes/` |
| Sync attribute behaviour | `Sandbox.Engine` | `Scene/Networking/Sync.cs`, `SyncFlags.cs` |
| RPC dispatch | `Sandbox.Engine` | `Scene/Networking/Rpc/` |
| Networking transport / lobbies | `Sandbox.Engine.Systems.Networking`, `Sandbox.NetCore` | `Systems/Networking/` |
| Hammer entity definition framework | `Sandbox.Engine` | `Editor/Hammer/HammerAttributes.cs` |
| Mapping tool (scene-mode mesh editing) | `game/addons/tools/Code/Scene/Mesh/Tools/` | `MeshTool.cs` and subtools |
| `MeshComponent` runtime | `Sandbox.Engine` | `Scene/Components/Mesh/MeshComponent.cs` |
| `PolygonMesh` data structure | `Sandbox.Engine` | `Scene/Components/Mesh/PolygonMesh.*.cs` |
| `MapInstance` runtime | `Sandbox.Engine` | `Scene/Components/Map/MapInstance.cs` |

## Internal engine systems

When you're changing how the engine works under the hood, not what it exposes.

| Topic | Project | Notes |
|---|---|---|
| User code compilation (Roslyn driver) | `Sandbox.Compiling` | Including code-archiving and the blacklist |
| Hotload / heap walking / state migration | `Sandbox.Hotload` | The deep-swap upgrader |
| API whitelist / sandbox verification | `Sandbox.Access` | IL inspection via Mono.Cecil |
| `TypeLibrary` / runtime reflection | `Sandbox.Reflection` | The catalogue used by Inspector, ActionGraph, snapshots |
| Source generators (`[Sync]`/`[Property]` codegen, etc.) | `Sandbox.Generator` | Roslyn analysers |
| `AppSystem` framework (module init order, ticking) | `Sandbox.AppSystem` | Boot-phase plumbing |
| Razor compiler | `Sandbox.Razor` | `.razor` → C# |
| Property binding / observation (UI) | `Sandbox.Bind` (`Links/`, `Proxy/`) | Used by Razor |
| Native interop wrappers | `Sandbox.Bind` (root) | Hand-written + InteropGen output |
| Solution scaffolding for user projects | `Sandbox.SolutionGenerator` | Generates user `.csproj`/`.sln` |
| Code upgraders (auto-migrate user code on API rename) | `Sandbox.CodeUpgrader` | Roslyn rewriters |

## Editor and tooling

When you're changing the editor itself or one of its built-in tools.

| Topic | Project | Notes |
|---|---|---|
| Inspector framework | `Sandbox.Tools` | `Inspector/` |
| `ControlWidget`s (per-type property editors) | `Sandbox.Tools` | `ControlWidget/` |
| Mapping (in-scene mesh editing) | `game/addons/tools/Code/Scene/Mesh/` | `MeshTool.cs` + subtools |
| Hammer integration | `Sandbox.Tools` | `MapEditor/` and `engine/Sandbox.Engine/Editor/Hammer/` |
| Built-in editor entities (lights, comments, etc.) | `Sandbox.Tools` | `MapEditor/HammerEntities/` |
| Shader Graph editor | `game/addons/tools/Code/ShaderGraph/` | |
| Action Graph editor | `game/editor/ActionGraph/Code/` | |
| Movie Maker | `game/editor/MovieMaker/Code/` | |
| Doo Editor (block-based scripting) | `game/editor/DooEditor/Code/` | |
| Model Editor | `Sandbox.Tools` | `ModelEditor/` |
| Mesh Editor (low-level) | `Sandbox.Tools` | `MeshEditor/` |
| Editor Apps (loader windows) | `Sandbox.Tools` | `Editor/` |
| Qt wrapper (native-to-managed UI) | `Sandbox.Tools` | `Qt/` |
| Code editor integration | `Sandbox.Tools` | `CodeEditor/` |

## Boot and runtime layout

When you're changing how the engine starts up or which executable does what.

| Topic | Project | Notes |
|---|---|---|
| .NET host bring-up | `engine/Launcher/Shared/Startup.cs` | Where CoreCLR is initialised |
| Per-launcher entry points | `engine/Launcher/<Variant>/Launcher.cs` | One per `sbox-*.exe` |
| Crash reporter | `engine/Launcher/CrashReporter/` | |
| Profiler | `engine/Launcher/SboxProfiler/` | |
| `AppSystem` boot ordering | `Sandbox.AppSystem` + per-system registrations | Adding a new module is a registration call |
| Configuration files (`.runtimeconfig.json`, etc.) | `game/` root + per-launcher | Edit alongside the launcher |

## Build pipeline tools

When you're changing how the engine itself builds.

| Topic | Project | Notes |
|---|---|---|
| `Bootstrap.bat` | Repo root | Calls SboxBuild |
| Build orchestrator | `engine/Tools/SboxBuild/` | Compile / shaders / content |
| Native binding generation | `engine/Tools/InteropGen/` | Reads `engine/Definitions/*.def` |
| Source generators (registered in `Sandbox.Generator`) | `engine/Tools/CodeGen/` | The standalone codegen helper |
| Shader compiler driver | `engine/Tools/ShaderCompiler/` | Wraps native shader tools |
| Shader preprocessor | `engine/Tools/ShaderProc/` | Shader Graph → HLSL |
| Asset cache builder | `engine/Tools/CreateGameCache/` | For standalone exports |
| Menu environment build | `engine/Tools/MenuBuild/` | |

## Native side

The `engine/bin/win64/*.dll` files come from the C++ Source 2 source tree, which is **not in this repo**. You typically don't modify these as a community contributor.

What you can do from this repo:

- Add a new C# wrapper for an existing native function — edit `engine/Definitions/<area>.def` and re-run InteropGen.
- Hand-write a wrapper when InteropGen's marshalling isn't enough — edit `Sandbox.Bind/<area>/` directly. There are existing patterns to copy.
- Update bindings after Facepunch publishes new native binaries — usually involves running InteropGen against updated `.def` files.

If you genuinely need a new native function, that's a Facepunch-side conversation; raise an issue.

## "I'm not sure where this lives"

The fastest way to find any user-facing API:

```bat
rg "class CharacterController\b" engine\
```

(or from your IDE's "find in files" against `engine/`). The first hit is the canonical location.

For attribute names: search for the attribute class definition.

```bat
rg "class SyncAttribute\b" engine\
```

For native bridge names: look in `engine/Definitions/` for `.def` files.

## Cross-cutting concerns

A few changes touch multiple projects. Common patterns:

**Adding a new property attribute (e.g., `[MyAttr]`):**

1. Define `MyAttrAttribute` in `Sandbox.System/Attributes/`.
2. If it needs runtime behaviour: handle it in `Sandbox.Reflection`'s `TypeLibrary` walk.
3. If it needs editor UI: add a `ControlWidget` in `Sandbox.Tools/ControlWidget/`.
4. Add tests in `Sandbox.Test`.

**Adding a new built-in component:**

1. Define the class in `Sandbox.Engine/Scene/Components/<group>/`.
2. Add `[Title]`, `[Icon]`, `[Category]` for the editor.
3. Mark `[Property]` fields appropriately.
4. If it pairs with native rendering/physics: add the `Sandbox.Bind` calls.
5. Document it under `docs/scene/components/reference/`.

**Adding a new networking primitive:**

1. The runtime side goes in `Sandbox.Engine/Scene/Networking/` or `Systems/Networking/`.
2. If it has an attribute: define in `Sandbox.System/Attributes/`.
3. If it needs codegen: add in `Sandbox.Generator/`.
4. Tests in `Sandbox.Test` (network-specific tests under `Scene/Networking/`).

**Adding a new shader feature:**

1. Native shader code: not in this repo (Facepunch-side).
2. C# shader registration in `engine/Sandbox.Engine/Resources/Shader/`.
3. Editor support: `Sandbox.Tools/ShaderGraph/` or `game/addons/tools/Code/ShaderGraph/`.

## Related Pages

- [Building from Source & Contributing](contributing.md)
- [Architectural Overview](../architecture.md) — the layer model
- [Engine Internals](index.md) — per-project deep dives
