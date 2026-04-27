---
title: Engine Concepts
icon: "🌲"
sources:
  - engine
updated: 2026-04-25
created: 2026-04-27
---

# Engine Concepts

The conceptual middle layer between the [Architectural Overview](../architecture.md) (one-page diagram) and the [Engine Internals](../engine-internals/index.md) (per-project reference). Read this when you want to understand *what* the engine actually is, in prose, before drilling into specific source files.

| For | Read |
|---|---|
| One-page overview with diagrams | [Architectural Overview](../architecture.md) |
| Conceptual prose overview (this page) | here |
| Per-`Sandbox.X` reference and source paths | [Engine Internals](../engine-internals/index.md) |
| Build / test / contribute workflow | [Building from Source](../engine-internals/contributing.md) |
| Decision tree: what to modify for a given change | [Where to Make Changes](../engine-internals/where-to-make-changes.md) |

## A concrete moment: you hit Ctrl+S on a `.cs` file

You're running a game in `sbox-dev.exe`. You alt-tab to your editor, change a line in a component's `OnUpdate`, and press Ctrl+S. By the time your hand leaves the key, the running game is executing the new code — without losing the position of your character, the contents of your inventory, or anything else in the live scene.

Three subsystems coordinate to make that happen, in milliseconds:

1. **`Sandbox.Compiling`** notices the file changed, invokes Roslyn against your `.sbproj`, and produces a new in-memory assembly. If the build fails, this whole sequence pauses until you fix it.
2. **`Sandbox.Access`** walks the new assembly's IL with `Mono.Cecil` and rejects it if it calls anything off the API whitelist (`System.IO.File.WriteAllText`, raw sockets, `Reflection.Emit`, etc.). Whitelisted? It's allowed to load.
3. **`Sandbox.Hotload`** performs the deep swap. It maps every type in the old assembly to its replacement, walks the live heap, instantiates new objects, copies state field-by-field, then redirects every reference in the running application from the old object to the new one. If only method *bodies* changed it takes the IL-patch fast path instead.

You do not see any of this. You see your character keep walking, with the new behaviour.

The rest of this page is what each of those subsystems is, in general — plus the boot sequence that brought them up, the runtime tick that calls them every frame, the virtual filesystem your game's assets sit in, and the sandboxing model that gives `Sandbox.Access` something to enforce.

## What s&box actually is

Beneath the surface, s&box is **C# wrapping Source 2**. The native engine — Source 2, written in C++ by Valve and extended by Facepunch — is responsible for the things native engines are good at: rendering through Vulkan or DirectX, simulating physics through Rubikon, mixing audio, managing the asset compiler, talking to GPUs and audio devices and the operating system. None of that is in this repo. The native binaries live under `game/bin/`.

Everything you can see and modify in `engine/` is the **C#-side runtime** sitting on top of that native foundation. It's a sequence of `.NET` projects that, between them, do four broad jobs:

1. **Bridge to native.** `Sandbox.Engine/Core/Interop/` exposes the C++ engine's public API as C# methods, generated from `.def` files by `engine/Tools/InteropGen` plus hand-written wrappers for the cases the generator can't express. `Sandbox.NetCore` is the tiny bootstrap shim that lets the native process load `Sandbox.Engine.dll` from an isolated assembly load context. (`Sandbox.Bind` is unrelated — it's the property data-binding system used by Razor UI.)
2. **Define the public game-dev API.** `Sandbox.Engine` is the bulk of this — `GameObject`, `Component`, `Scene`, `Rigidbody`, `MapInstance`, `[Sync]`, RPCs, the lifecycle methods every game dev overrides. When a game dev writes `using Sandbox;`, they're consuming this surface.
3. **Manage the user's code at runtime.** `Sandbox.Compiling` invokes Roslyn against the user's `.sbproj`. `Sandbox.Hotload` walks the live heap and migrates state when the user saves a file. `Sandbox.Reflection` provides a runtime catalogue (`TypeLibrary`) that the inspector, ActionGraph, and the snapshot system all read. `Sandbox.Access` verifies the user's compiled assembly doesn't call blacklisted .NET APIs before letting it run.
4. **Provide the editor.** `Sandbox.Tools` is the editor itself: the Inspector, the Mapping tool, Hammer integration, Shader Graph, Movie Maker, Action Graph editor, Qt wrapping for native UI. None of it ships with `sbox.exe` — only `sbox-dev.exe` loads it.

A game dev only ever sees layer 2. A contributor changing core engine behaviour usually works in layers 2 or 3. Layers 1 and 4 are touched by people doing native bindings or editor tooling specifically.

## The boot sequence, conceptually

Each `sbox-*.exe` (the variants in `engine/Launcher/`) follows the same boot pattern, defined in `engine/Launcher/Shared/Startup.cs`:

1. The native process starts up.
2. The .NET host (CoreCLR) is brought up inside the native process, not the other way around. The launcher runs from native; only after CoreCLR is hosted does C# code start executing.
3. The launcher mounts the baseline filesystems (`game/core/`, addons referenced in the launch config) into the virtual filesystem.
4. **`AppSystem`** modules are initialised in dependency order. Each subsystem (Render, Audio, Physics, Networking, Input, Hotload, etc.) is an `AppSystem` class with a `Init()` and a `RunFrame()`. Their boot order is determined by dependency declarations, not source order.
5. The launcher hands control to whatever runs the chosen mode — the editor's main window for `sbox-dev.exe`, the standalone game runtime for `sbox.exe`, the headless server loop for `sbox-server.exe`.
6. From this point, the native engine ticks the main loop. Each frame, native invokes `Sandbox.Engine.Core.EngineLoop.RunFrame()`, which dispatches to active scenes and through them to user `Component`s.

This means **C# isn't the entry point**. The native process is. C# is hosted, ticked, and unloaded by native. When you crash inside C#, the native crash reporter catches it.

## The runtime tick

Inside `Sandbox.Engine.Core.EngineLoop.RunFrame`, each frame:

1. Pre-frame work — input dispatch, timer ticks, `AppSystem.RunFrame` for systems that need pre-tick work.
2. The active `Scene` ticks — it walks its GameObject tree, calling lifecycle methods on enabled components: `OnFixedUpdate` at fixed timestep, `OnUpdate` at framerate, `OnPreRender` just before submission.
3. Physics steps. The engine wraps Rubikon; `Scene.PhysicsWorld` is the C# handle. Physics events (`IScenePhysicsEvents.PrePhysicsStep`/`PostPhysicsStep`) fire here.
4. Render submission — the engine walks the scene's renderable components and submits draw calls to the native render system.
5. Networking — snapshots are sent, RPCs are dispatched.
6. Post-frame `AppSystem.RunFrame` for systems that need post-tick work.

This loop is **single-threaded by convention**. The Source 2 scene graph and renderer require main-thread submission. Engine code can spawn `Task`s for offloading, but anything that touches scene state has to marshal back to the main thread first. Game devs writing `Component`s don't need to think about this — their `OnUpdate` already runs on the main thread.

## Hot reloading, conceptually

The thing s&box does that most engines don't: **swap user assemblies live, while the game runs, preserving state.**

When a user saves a `.cs` file, three projects coordinate:

1. **`Sandbox.Compiling`** — invokes Roslyn against the user's project. Output is a new in-memory assembly. If compilation fails, hotload pauses; nothing else happens until the user fixes the errors.
2. **`Sandbox.Access`** — runs assembly verification on the new assembly. Walks IL with `Mono.Cecil`, ensures no blacklisted APIs are called. If the verification fails, the assembly is rejected and hotload pauses.
3. **`Sandbox.Hotload`** — performs the deep swap. It maps every type in the old assembly to its replacement in the new one, walks the live heap, instantiates the new types, and copies state field-by-field from old instances to new ones. Then it uses pointer manipulation to redirect every reference held anywhere in the running application from the old object to the new one.

After the swap, user-side `OnRefresh` callbacks fire so caches and re-derivations can run. `OnAwake` and `OnStart` do not re-run — those are first-creation hooks, and from the live state's perspective, the components haven't been re-created.

There's also a fast path: **IL hotload**. When only method *bodies* changed (no type-shape changes), `Sandbox.Hotload` patches the IL in place rather than walking the heap. This is what makes most edits feel instantaneous.

The deep-swap path has consequences contributors need to know about. Static caches that hold `Type` references prevent the corresponding objects from being collected and replaced. Default field initialisers don't re-run on existing instances (the old field value is copied across). Lambdas with reordered closure variables can fail to migrate. The [Hotloading developer guide](../code/code-basics/hotloading.md) catalogues the user-facing pitfalls; the [Sandbox.Hotload page](../engine-internals/sandbox-hotload.md) documents the internals.

## The virtual filesystem and mounting

`FileSystem` in s&box isn't direct disk I/O. It's a **virtual filesystem** layered over a stack of mounted packages. When you write `FileSystem.Mounted.OpenRead("models/citizen/citizen.vmdl")`, the engine consults its mount stack — addon by addon, in reverse priority order — and serves the first match.

Mounts come from several sources:

- **`game/core/`**, mounted at boot — engine baseline assets every game gets.
- **`game/addons/<name>/`** referenced in your `.sbproj` — the foundational `base`, `citizen`, `menu`, `tools` addons plus anything else.
- **Your project**, sitting at the top of the stack so its overrides win.
- **External games** loaded through `engine/Mounting/Sandbox.Mounting.*` — GoldSrc, NS2, Quake. These are special: they don't ship their assets in s&box's native formats, so the mount provider intercepts asset reads and converts on demand. Quake's `.bsp` becomes a Source 2 scene, its palette-based textures become RGBA `Texture` instances, all transparently.

Writing a new external mount means subclassing `BaseGameMount` (in `Sandbox.Mounting`), declaring an `Ident` and `Title`, implementing `Initialize` to detect whether the game is installed, and implementing `Mount` to register the assets. See [Creating Mounts](../systems/mounting/creating-mounts.md) for the user-facing how-to and [Mounting Internals](../engine-internals/mounting.md) for the engine side.

## Sandboxing, conceptually

s&box runs C# code from anyone on the internet — a player joining a server downloads that server's game code and the engine compiles and runs it. That makes the **API whitelist** load-bearing.

The whitelist is a list of allowed .NET methods, types, and namespaces. Everything not on the list is blocked. The most-asked-about blocks: `System.IO` (use `FileSystem` instead), raw sockets (use `Sandbox.Http` or `Sandbox.WebSocket`), `Reflection.Emit` (use `Sandbox.Generator` source generators), `System.Threading.Thread` (use `Task`).

Enforcement happens at two layers:

1. **Compile time.** The Roslyn analyser running inside `Sandbox.Compiling` flags whitelist violations during the user's edit. The user sees them in the editor console as errors.
2. **Load time.** `Sandbox.Access` walks the compiled assembly's IL with Mono.Cecil before letting it execute. This catches assemblies that bypassed the source-time check (someone shipped a precompiled DLL into the package).

The compile-time check exists for good error messages; the load-time check exists for security. Both must pass.

## Where to go next

If you're orienting yourself: [Architectural Overview](../architecture.md) is the diagram, this page is the prose, and [Engine Internals](../engine-internals/index.md) is the reference.

If you're already in this section, the conceptual subpages drill into specific areas:

- [Definitions](definitions.md) — the `engine/Definitions/` directory and how it feeds InteropGen.
- [Launcher](launcher.md) — what each `sbox-*.exe` does and how the boot sequence is shared.
- [Mounting](mounting.md) — the conceptual model for how packages get attached to the VFS.
- [Sandbox.Access](sandbox-access.md) — the security boundary in detail.
- [Sandbox.AppSystem](sandbox-appsystem.md) — the module-lifecycle framework that boots subsystems.
- [Sandbox.Bind](sandbox-bind.md) — the property data-binding system used by Razor UI (separate from the native interop).

For everything else (`Sandbox.Compiling`, `Sandbox.Hotload`, `Sandbox.Reflection`, `Sandbox.Tools`, `Sandbox.Engine`), go directly to the per-project pages under [Engine Internals](../engine-internals/index.md).
