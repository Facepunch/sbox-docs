---
title: "Glossary"
icon: "📖"
created: 2026-04-25
updated: 2026-04-25
---

# Glossary

Short definitions of terms you'll hit while working in s&box. Each entry is one or two sentences plus a link to the deeper page when there is one.

## How to use this glossary

There are three ways in, depending on what you have:

1. **You're browsing a topic** — start with the [By Domain](#by-domain) section below. Terms are grouped by area (Scene, Networking, Physics, Audio, Rendering, Mapping, etc.). Use this when you don't know the exact word but know the area.
2. **You came from another engine** — jump to [Vocabulary from other engines](#vocabulary-from-other-engines). Unity, Unreal, and generic terms are translated to their s&box equivalents.
3. **You have a specific term in mind** — Ctrl-F this page, or jump to the [Alphabetical Index](#alphabetical-index) at the bottom for the full A-Z list.

> **Looking up a term you'd use elsewhere?** s&box has its own vocabulary. The [vocabulary mapping section](#vocabulary-from-other-engines) translates Unity, Unreal, and generic terms into s&box equivalents. Use it when "level," "script," "actor," etc. don't seem to find anything.

---

## By Domain

Terms grouped by what part of s&box they belong to. One sentence each. Click through to the alphabetical index for the longer definitions.

### Scene & Components

The runtime tree of GameObjects and the Component lifecycle that drives game logic.

- **[GameObject](#g)** — A thing in the scene; holds transform, tags, components, children.
- **[Component](#c)** — The unit you write game logic in; attached to a GameObject.
- **[Component Lifecycle](#c)** — `OnAwake` → `OnEnabled` → `OnStart` → `OnUpdate` / `OnFixedUpdate` → `OnDisabled` → `OnDestroy`.
- **[`OnAwake`](#o)** — First lifecycle method; siblings may not have woken yet.
- **[`OnStart`](#o)** — Lifecycle method called once when the component is fully active; safe to query siblings.
- **[`OnEnabled` / `OnDisabled`](#o)** — Fire on each enabled/disabled flip; subscribe/unsubscribe here.
- **[`OnUpdate`](#o)** — Per-frame game-logic hot path.
- **[`OnFixedUpdate`](#o)** — Fixed-timestep (~50 Hz) update; for physics-driving code.
- **[`OnPreRender`](#o)** — Called per frame just before render; never on dedicated servers.
- **[`OnDestroy`](#o)** — Final lifecycle method; release external resources here.
- **[`OnLoad`](#o)** / **[`async OnLoad`](#a)** — Async lifecycle method; loading screen waits for it.
- **[`OnRefresh`](#o)** — Called after hotload or deserialization; rebuild caches here.
- **[`OnValidate`](#o)** — Called when an inspector value changes; clamp/enforce invariants here.
- **[`Clone()`](#c)** — Duplicates a GameObject and descendants.
- **[`GameObject.IsValid()`](#g)** — Returns false after destruction; check stored references.
- **[`GameObject.Tags`](#g)** / **[Tags](#t)** — String labels on a GameObject; inherited by children.
- **[`GameObjectSystem`](#g)** — Scene-level system running at specific frame phases.
- **[Execution Order](#e)** — Order sibling components run their lifecycle methods.
- **[Prefab](#p)** — Saved GameObject hierarchy that can be instantiated multiple times.
- **[Scene](#s)** — The runtime tree of GameObjects you're playing in.
- **[`ISceneStartup`](#i)** — Scene event interface for `OnHostInitialize` / `OnClientInitialize`.
- **[`Component.IPressable`](#c)** — Component interface for "use" key presses.
- **[`Component.ITriggerListener`](#c)** — Receives `OnTriggerEnter` / `OnTriggerExit`.
- **[`Component.ExecuteInEditor`](#c)** — Marker telling the engine to run lifecycle in edit mode.
- **[`FindMode`](#f)** — Enum filter for `Components.Get<T>(FindMode)` lookups.

### Networking

Multiplayer state replication, ownership, RPCs, and lobbies.

- **[`[Sync]`](#s)** — Replicates a property's value across the network.
- **[`SyncFlags`](#s)** — Flags for `[Sync(SyncFlags.X)]` (`Query`, `FromHost`, `Interpolate`).
- **[`[Change(nameof(method))]`](#c)** — Reacts to a synced property changing.
- **[`[Rpc.Broadcast]`](#r)** — Method runs on every peer when called.
- **[`[Rpc.Host]`](#r)** — RPC variant that only runs on the host.
- **[`[Rpc.Owner]`](#r)** — RPC variant that only runs on the GameObject's owner.
- **[`Rpc.Caller`](#r)** — Inside an RPC, the connection that invoked it.
- **[`Connection`](#c)** — A connected client (or the host); `Connection.Local`, `.Host`, `.All`.
- **[`Networking`](#n)** — Static class: `IsHost`, `IsClient`, `CreateLobby`, `Connect`.
- **[Network Helper](#n)** — Drop-in lobby helper component in the `base` addon.
- **[`NetworkSpawn()`](#n)** — Registers a GameObject with the network.
- **[`NetworkMode`](#n)** — `Object` (synced), `Snapshot` (initial only), `Never` (local-only).
- **[`GameObject.Network`](#g)** — Networking surface (NetworkSpawn, IsOwner, Refresh, etc.).
- **[`IsProxy`](#i)** — `true` when this client is *not* the owner; common networking guard.
- **[Owner](#o)** — Peer with authority over a networked GameObject.
- **[OwnerTransfer](#o)** — `Fixed`, `Takeover`, `Request` — who can take ownership.
- **[Host](#h)** — Peer that owns the canonical world state in a multiplayer session.
- **[Lobby](#l)** / **[`LobbyConfig`](#l)** — A multiplayer session and its config.
- **[Snapshot](#s)** — Serialised state of all networked GameObjects sent on join.
- **[`NetList<T>`](#n)** / **[`NetDictionary<K,V>`](#n)** — Networked collections for `[Sync]`.
- **[Dedicated Server](#d)** — Headless multiplayer server (`sbox-server.exe`).
- **[Steam P2P](#s)** — Transport powering casual multiplayer through Steam relay.
- **[`IGameObjectNetworkEvents`](#i)** — Per-GameObject network event interface.
- **[Proxy](#p)** — "The local peer is not the owner of this object."

### Physics

Collision, simulation, traces, joints.

- **[`Rigidbody`](#r)** — Physics-simulating component with mass, gravity, velocity.
- **[`PhysicsType`](#p)** — `Static` / `Kinematic` / `Dynamic` mode for a Rigidbody.
- **[Kinematic](#k)** — Rigidbody whose position is set by code, not simulated.
- **[`Collider`](#c)** — Base type for collision shapes (Box/Sphere/Capsule/Hull/Mesh).
- **[`IsTrigger`](#i)** — Collider flag: detect entry/exit without blocking movement.
- **[Trigger](#t)** — A collider with `IsTrigger = true`.
- **[Joints](#j)** — Components connecting two Rigidbodies (Hinge, Slider, Ball, etc.).
- **[`CharacterController`](#c)** — Kinematic player-movement component.
- **[`CharacterController.Punch(force)`](#c)** — Velocity burst (jumps, impulses).
- **[`Jump(force)`](#j)** — `PlayerController.Jump` — vertical velocity burst.
- **[Trace](#t)** / **[`SceneTrace` / `SceneTraceResult`](#s)** — Ray/sphere/box/capsule cast through the physics world.
- **[Physics World](#p)** — The collision/simulation context for a Scene.
- **[`IScenePhysicsEvents`](#i)** — `PrePhysicsStep` / `PostPhysicsStep` hooks.
- **[`ModelPhysics`](#m)** — Multi-body physics from a model's joints (ragdolls).
- **[Rubikon](#r)** — Source 2's underlying physics engine.

### Audio

Sound events, mixers, spatial audio, DSP, voice.

- **[`SoundEvent`](#s)** — `.sound` asset wrapping audio with playback rules.
- **[`SoundPointComponent` / `SoundBoxComponent`](#s)** — Continuous attached audio.
- **[`SoundscapeTrigger`](#s)** — Switches background ambience by volume.
- **[Mixer](#m)** — Audio bus for routing and global volume.
- **[`DSPVolume`](#d)** — Real-time reverb/low-pass when listener enters volume.
- **[`AudioListener`](#a)** — Overrides where the local client hears from.
- **[`MusicPlayer`](#m)** — Streaming-audio player (separate from sound effects).
- **[`LipSync`](#l)** — Drives morph-target weights from sound spectrum.
- **[Voice](#v)** — Captures microphone, replicates to peers.

### Rendering

Cameras, materials, models, lighting.

- **[`CameraComponent`](#c)** — Viewpoint for rendering a scene.
- **[`FieldOfView`](#f)** — `CameraComponent.FieldOfView`, in degrees.
- **[`ModelRenderer`](#m)** — Draws a static `Model` at the GameObject's transform.
- **[`SkinnedModelRenderer`](#s)** — Animated skeletal models; drives bones, AnimationGraph.
- **[`Model.Load(path)`](#m)** — Static loader for a `Model` from a `.vmdl`.
- **[Material](#m)** — `.vmat` resource describing a surface's render parameters.
- **[`SceneObject`](#s)** — Internal native render object produced by renderer components.
- **[`AmbientLight`](#a)** — Component adding non-directional ambient light.
- **[`IndirectLightVolume`](#i)** — Baked indirect lighting volume.
- **[`Decal`](#d)** — Projects a texture onto surfaces.
- **[`HighlightOutline`](#h)** — Selection-style outline render.
- **[`VideoPlayer`](#v)** — Streams a video file/URL; exposes frame as Texture.
- **[`VideoWriter`](#v)** — Encodes RGBA frames into a video file.
- **[`VerletRope`](#v)** — Verlet-integrated cable/rope simulation.
- **[`DebugOverlay`](#d)** — Runtime visual debug primitives (sphere, line, text).
- **[`Gizmo.Draw`](#g)** — Editor-time drawing API (selection handles, debug lines).

### Mapping & Maps

Level authoring (Mapping tool, Hammer) and runtime map loading.

- **[Map](#m)** — A loadable level (`.vpk`, `.scene`, or package ident).
- **[`MapInstance`](#m)** — Runtime component that loads and manages a map.
- **[Map Entity](#m)** — A placeable thing inside a Hammer map.
- **[`MapObjectComponent`](#m)** — Internal component on map-entity GameObjects.
- **[`UseMapFromLaunch`](#u)** — `MapInstance` flag honouring `LaunchArguments.Map`.
- **[Scene Map](#s)** — A `.scene` file used as a map.
- **[Scene Mapping](#s)** — In-scene level-authoring workflow.
- **[Mapping (tool)](#m)** / **[`MeshTool`](#m)** — In-scene mesh-editing toolset.
- **[Block Editor](#b)** — Mapping tool's primitive editor for box geometry.
- **[Brushwork](#b)** — Hammer-era term for solid map geometry.
- **[Displacement](#d)** — Sculptable height-mapped surface in Mapping.
- **[`MeshComponent`](#m)** — Owns a `PolygonMesh`; produced by Mapping.
- **[`PolygonMesh`](#p)** — Editable half-edge mesh data type.
- **[`HammerMesh`](#h)** — Mesh component for Hammer-authored maps.
- **[Hammer](#h)** — Legacy `.vmap`-based level editor (being deprecated).
- **[`[HammerEntity]`](#h)** — Marks a class as a placeable Hammer entity.
- **[`[FGDType]`](#f)** — Hints which Hammer editor widget to use for a property.
- **[`.vmap`](#v)** — Hammer's source map file (compiles to `.vpk`).
- **[`.vpk`](#v)** — Compiled map file.
- **[`Terrain`](#t)** — Large-scale heightmap-based terrain component.
- **[NavMesh](#n)** / **[`NavMeshAgent`](#n)** — AI pathfinding mesh and agent.

### Editor & Tooling

The authoring environment, inspector, action graph, UI markup.

- **[Editor](#e)** (`sbox-dev.exe`) — Full authoring environment.
- **[`[EditorTool]`](#e)** — Registers a class as a top-level toolbar tool.
- **[Inspector](#i)** — Editor panel showing component `[Property]` fields.
- **[`[Property]`](#p)** — Exposes a property to the inspector and ActionGraph.
- **[`[Range]`](#r)** — Numeric constraint, optional slider.
- **[`[Title]`](#t)** — Inspector display-name override.
- **[`[Group]`](#g)** — Collapsible group in the inspector.
- **[`[ToggleGroup]`](#t)** — Bool-toggleable property group.
- **[`[Order(int)]`](#o)** — Inspector display order.
- **[`[Hide]`](#h)** — Hides a class/property from the inspector.
- **[`[Advanced]`](#a)** — Hides until "show advanced" is toggled.
- **[`[Feature]` / `[FeatureEnabled]`](#f)** — Toggleable inspector tab.
- **[`[Icon]`](#i)** — Material-icon name for the inspector header.
- **[`[ShowIf]` / `[HideIf]`](#s)** — Conditional property visibility.
- **[`HelpUrl`](#h)** — Class attribute pointing to documentation.
- **[`[FilePath]`](#f)** — File-picker for a string property.
- **[`[KeyProperty]`](#k)** — "Key" property for compact custom-type editing.
- **[`WideMode`](#w)** — Render editor below label, full width.
- **[`[Expose]`](#e)** — Makes a class visible to ActionGraph and inspector.
- **[ActionGraph](#a)** — Visual scripting saved as `.action` assets.
- **[Pulse / Data nodes](#p)** — ActionGraph execution-flow vs value-evaluating nodes.
- **[Razor](#r)** — HTML-like markup language for UI (`.razor` files).
- **[`PanelComponent`](#p)** — Razor-driven UI component.
- **[`WorldPanel`](#w)** — `PanelComponent` rendered in 3D space.
- **[`HudPainter`](#h)** — Immediate-mode 2D drawing API for HUDs.
- **[Qt](#q)** — C++ UI framework underlying the editor.

### Code & Compilation

How your `.cs` becomes a running, hot-loadable assembly.

- **[Hotload](#h)** — Recompile-and-replace cycle that migrates live state.
- **[IL Hotload](#i)** — Fast-path that swaps method bodies without heap walking.
- **[`[SkipHotload]`](#s)** — Marks a class/field for the upgrader to leave alone.
- **[Roslyn](#r)** — .NET compiler used by `Sandbox.Compiling`.
- **[`Sandbox.Compiling`](#s)** — Runtime Roslyn compiler producing the hot-loadable assembly.
- **[`Sandbox.Hotload`](#s)** — Deep-swap upgrader that walks the heap.
- **[`Sandbox.Reflection`](#s)** — Engine reflection layer (`TypeLibrary`).
- **[`TypeLibrary`](#t)** — Reflection cache shared by inspector, ActionGraph, snapshots.
- **[Upgrader](#u)** — Roslyn rewriter that auto-migrates user C# on API changes.
- **[API Whitelist](#a)** — Standard .NET APIs allowed in sandboxed user code.
- **[Sandbox](#s)** — The `Sandbox` namespace and the security boundary.
- **[`Application`](#a)** — Static class with global runtime info.
- **[`Event`](#e)** — Three different things in s&box (project, attribute, delegates).
- **[`GameTask`](#g)** — Engine-aware async helpers.
- **[`Library("classname")`](#l)** — Stable identifier for a class.
- **[InteropGen](#i)** — Tool that generates `Sandbox.Bind` P/Invoke wrappers.
- **[Interceptor](#i)** — Mounting component handling a specific file format.

### Engine Architecture

Engine-internal modules, native bridge, mounting.

- **[`AppSystem`](#a)** / **[`Sandbox.AppSystem`](#s)** — Module-lifecycle framework.
- **[`Sandbox.Bind`](#s)** — Native interop and Razor property binding.
- **[`EngineLoop`](#e)** — Managed main-loop entry point called from native.
- **[`engine/`](#e)** — Engine source tree shipped in this repo.
- **[`engine-internals/`](#e)** — Wiki section documenting each `Sandbox.*` project.
- **[Source 2](#s)** — The native C++ engine s&box runs on.
- **[Mount](#m)** — Game/asset mounted into the virtual filesystem.
- **[`FileSystem`](#f)** — Virtual filesystem (`Mounted`, `Data`).
- **[`Application`](#a)** — `IsStandalone`, `IsHeadless`, `Quit()`.
- **[Standalone](#s)** — Self-contained game export with its own `.exe`.
- **[Forge](#f)** — THEMIS orchestrator that runs wiki-generation agents.

### Project & Build

Project files, packages, build pipeline.

- **[`.sbproj`](#s)** — Project file (JSON metadata).
- **[`.scene`](#s)** — Scene file (JSON, version-controlled).
- **[Package](#p)** — Bundled addon, game project, or standalone export.
- **[Addon](#a)** — Package that extends the engine or another game.
- **[`base` addon](#b)** — Foundational addon almost every game depends on.
- **[Avatar](#a)** — Default Citizen character used across many games.
- **[Citizen](#c)** — Default character model and clothing/animation system.
- **[Build Pipeline](#b)** — Compile-and-package process (Roslyn → `Sandbox.Compiling` → `SboxBuild`).
- **[`Bootstrap.bat`](#b)** — Run once to download native binaries.
- **[`LaunchArguments`](#l)** — Static class capturing engine-launch args.
- **[Cookbook](#c)** — How-to recipes section of the wiki.

### Math & Time

Math types and time helpers.

- **[`Vector3`](#v)** — 3-component float vector (Z-up).
- **[`Rotation`](#r)** — Quaternion-backed rotation type.
- **[`Angles`](#a)** — Pitch/yaw/roll as three floats.
- **[`BBox`](#b)** — Axis-aligned bounding box.
- **[`MathX`](#m)** — Static math helpers (`Lerp`, `Remap`, `Clamp`, `SmoothStep`).
- **[`WorldPosition` / `WorldRotation` / `WorldScale`](#w)** — GameObject transform in world space.
- **[`Time.Delta`](#t)** — Seconds since the previous `OnUpdate`.
- **[`Time.FixedDelta`](#t)** — Time per fixed-update tick.
- **[`Time.Now`](#t)** — Current scene time in seconds.
- **[`Time.TimeScale`](#t)** — Global pause/slow-mo.

### Misc / Disambiguation

Words that overload elsewhere — what they mean here.

- **[Actor](#a)** — Not an s&box term. Closest: `GameObject`.
- **[Level](#l)** — Not an s&box term. Use `Map` (asset) or `Scene` (runtime).
- **[Script](#s)** — Not an s&box term. Logic lives in `Component`s.
- **[World](#w)** — Not standard. Runtime is `Scene`; on-disk is `Map`.
- **["Local"](#concepts-that-dont-belong-on-a-single-page)** — local machine. `Connection.Local` vs `LocalPosition`.
- **["Owner" vs "Host"](#concepts-that-dont-belong-on-a-single-page)** — per-object authority vs per-session.
- **["Mounted"](#concepts-that-dont-belong-on-a-single-page)** — added to the virtual filesystem.
- **["Networked" GameObject](#concepts-that-dont-belong-on-a-single-page)** — registered via `NetworkSpawn`.
- **["Static" / "Kinematic" / "Dynamic"](#concepts-that-dont-belong-on-a-single-page)** — Rigidbody modes.

---

## Alphabetical Index

Use this when you have a specific term in mind. Letter jump:

[A](#a) · [B](#b) · [C](#c) · [D](#d) · [E](#e) · [F](#f) · [G](#g) · [H](#h) · [I](#i) · [J](#j) · [K](#k) · [L](#l) · [M](#m) · [N](#n) · [O](#o) · [P](#p) · [Q](#q) · [R](#r) · [S](#s) · [T](#t) · [U](#u) · [V](#v) · [W](#w)

## A

**Actor** — Not an s&box term. The closest equivalent is **[GameObject](#g)** — but s&box's GameObject is much lighter than an Unreal Actor (no class hierarchy to inherit from). See the [Unreal → s&box mapping](#unreal--sbox).

**Addon** — A package that extends the engine or another game. The `base`, `citizen`, `menu`, and `tools` addons ship with s&box; community addons live in `game/addons/`. See [Addons & Extensions](engine-internals/addons/index.md).

**`[Advanced]`** — Property attribute that hides a property from the inspector unless the user explicitly toggles "show advanced." For knobs that aren't meant for typical users.

**`AmbientLight`** — Component that adds non-directional ambient light to a scene. See [Ambient Light](scene/components/reference/ambientlight.md).

**`Angles`** — Math type representing pitch/yaw/roll as three floats. Convert to a `Rotation` via `.ToRotation()`. Easier than quaternions when you don't need composition.

**ActionGraph** — Visual scripting system for game logic. Saved as `.action` assets, executed via reflection. See [ActionGraph](systems/actiongraph/index.md). Some legacy node forms are `[Obsolete]` (constant nodes); the system itself is supported.

**`AppSystem`** — The engine's module-lifecycle framework. Every initialisation phase (Render, Physics, Networking, etc.) is an `AppSystem` registered in dependency order. You rarely interact with these directly. See [Sandbox.AppSystem](engine-internals/sandbox-appsystem.md).

**API Whitelist** — The list of standard .NET APIs allowed in sandboxed user code. `System.IO`, raw sockets, `Reflection.Emit`, etc. are blocked; the engine provides safe equivalents. See [API Whitelist](code/code-basics/api-whitelist.md).

**`Application`** — Static class with global runtime info: `Application.IsStandalone`, `Application.IsHeadless`, `Application.Quit()`.

**`async OnLoad`** — A `Component`'s `OnLoad` override can be async (`Task OnLoad()`). The loading screen waits for it before the scene starts. Used for downloading data or generating procedural content.

**`AudioListener`** — Component that overrides where the local client hears audio from. Default is the camera; you can attach it to a player's head for first-person.

**Avatar** — The default Citizen character used across many s&box games. Players customise it via the menu's avatar editor and games can opt in to using it. See [Avatar](systems/avatar/index.md) and [Citizen Addon](build-games/addons/citizen/index.md).

## B

**`base` addon** — The foundational addon almost every game depends on: shared UI controls, networking helpers, common components, default styles. See [Base Addon](./engine-internals/addons/base-code.md).

**`BBox`** — Axis-aligned bounding box. `BBox.FromPositionAndSize(...)`, `bbox.Contains(point)`, `bbox.Mins`/`Maxs`.

**Block Editor** — The Mapping tool's primitive editor for blocking out box-shaped geometry. Drag in 3D space to create a `MeshComponent` with a box `PolygonMesh`. See [Scene Mapping](scene/scene-mapping/index.md).

**`Bootstrap.bat`** — The script you run once to download native binaries and prepare the engine for local builds.

**Brushwork** — Hammer-era term for solid map geometry built from convex volumes. In Scene Mapping, the modern equivalent is editing a `PolygonMesh` via `MeshComponent`.

**Build Pipeline** — The compile-and-package process: Roslyn compiles your `.cs`, `Sandbox.Compiling` produces a hot-loadable assembly, optional native steps run via `engine/Tools/SboxBuild`. See [Tools (Engine Tooling)](engine-internals/tools/index.md).

## C

**`CameraComponent`** — The viewpoint for rendering a scene. `Scene.Camera` returns the active one. `FieldOfView`, `BackgroundColor`, `ScreenPixelToRay()`, `ScreenRect`. See [Camera Component](scene/components/reference/cameracomponent.md).

**`CharacterController`** — Kinematic player-movement component. Doesn't simulate forces; you set its `Velocity` and call `Move()`. Convenient for FPS/platformer characters where you want frame-perfect control. See [Character Controller](scene/components/reference/charactercontroller.md).

**`[Change(nameof(method))]`** — Property attribute that triggers `method(oldValue, newValue)` whenever the property changes. Pairs with `[Sync]` to react to networked value changes.

**Citizen** — The default character model and accompanying clothing/animation system. See [Citizen Addon](build-games/addons/citizen/index.md) and [Avatar](systems/avatar/index.md).

**`Clone()`** — `GameObject.Clone()` duplicates a GameObject and its descendants. Multiple overloads accept a `Transform`, parent, and starting-enabled flag. For prefabs, `Clone(prefabPath, ...)` is the static form.

**`Collider`** — Base type for collision shapes (`BoxCollider`, `SphereCollider`, `CapsuleCollider`, `HullCollider`, `MeshCollider`). Pairs with `Rigidbody` for dynamic physics; alone, defines static collision. See [Colliders](systems/physics/colliders.md).

**Component** — A piece of behaviour or data attached to a GameObject. The fundamental unit you write s&box code in. See [Components](scene/components/index.md).

**`Component.ExecuteInEditor`** — Marker interface telling the engine to run this component's `OnEnabled`/`OnUpdate` even in edit mode (when the scene isn't playing). Used by editor-only components like `MeshComponent`.

**`Component.IPressable`** — Implement this on a component to make it respond to "use" key presses on raycast-hit objects.

**`Component.ITriggerListener`** — Implement this on a component to receive `OnTriggerEnter` / `OnTriggerExit` callbacks from a sibling collider with `IsTrigger = true`.

**Component Lifecycle** — The sequence of method calls the engine makes on a component: `OnAwake` → `OnEnabled` → `OnStart` → `OnUpdate` (per-frame) / `OnFixedUpdate` (per-tick) → `OnDisabled` → `OnDestroy`. See [Component Lifecycle](scene/components/component-lifecycle.md).

**`Connection`** — Represents a connected client (or the host) in a multiplayer session. `Connection.Local` is you. `Connection.Host` is the host. `Connection.All` enumerates all connected peers.

**Cookbook** — The collection of how-to recipes at [docs/how-to/](how-to/index.md). Goal-oriented, copy-pasteable, brief.

**`CharacterController.Punch(force)`** — Applies a velocity burst to the controller — used for jumps and impulses. Different from a `Rigidbody`'s `ApplyImpulse` because the controller is kinematic.

## D

**`DebugOverlay`** — Static class for runtime visual debug primitives. `DebugOverlay.Sphere(center, radius, ...)`, `DebugOverlay.Line(a, b, ...)`, `DebugOverlay.ScreenText(pos, text, ...)`, `DebugOverlay.Text(worldPos, text, ...)`. All take optional `duration` and `overlay` flags.

**`Decal`** — Component that projects a texture onto surfaces in the world. Decay rules, lifetime control, multiple projection axes. See [Decal](scene/components/reference/decal.md).

**Dedicated Server** — Headless multiplayer server (`sbox-server.exe`) without rendering or audio. Runs your game's server-side logic and arbitrates between clients. See [Dedicated Servers](systems/networking-multiplayer/dedicated-servers/index.md).

**`DSPVolume`** — Component that applies real-time DSP (reverb, low-pass) when the `AudioListener` is inside its volume. Used for caves, water, indoor reverb. See [DSP Volume](systems/audio/dsp-volume.md).

**Displacement** — A smooth height-mapped surface. The Mapping tool's Displacement subtool turns a flat face into a sculptable terrain-like surface. Smaller scope than a full `Terrain`.

## E

**Editor** (`sbox-dev.exe`) — The full authoring environment — scene editor, inspector, console, Mapping tool, Hammer, Shader Graph, Movie Maker. Loaded only by `sbox-dev`; stripped from standalone exports. See [Editor](editor/index.md).

**`[EditorTool]`** — Attribute that registers a class as a top-level tool in the scene editor toolbar. Used by `MeshTool` (Mapping), Hammer, etc.

**`EngineLoop`** — `Sandbox.Engine.Core.EngineLoop` — the managed main-loop entry point that the native engine calls each frame. Drives scene ticks, physics, render. You don't call into it directly.

**`engine/`** — The engine source tree. Ships in this repo. Read it whenever a documentation page is unclear; canonical truth lives there.

**`engine-internals/`** — The wiki section documenting each `Sandbox.*` engine project. Useful for engine contributors; usually not needed by game devs.

**`Event`** — Three different things in s&box, easy to confuse:
1. The `Sandbox.Event` project (engine-internal event bus).
2. `[Event("name")]` — attribute that hooks a method to a global engine event like `"hotload"`.
3. `Action` properties on components like `MapInstance.OnMapLoaded` — those are just C# delegates, not the event bus.

**Execution Order** — The order in which sibling components run their lifecycle methods. Default is unspecified; use a `GameObjectSystem` with a stage and order if you need control. See [Execution Order](scene/components/execution-order.md).

**`[Expose]`** — Marker that makes a class visible to ActionGraph and the inspector for reflection-based UI. Most user-facing components are implicitly exposed.

## F

**`[Feature("name")]` / `[FeatureEnabled("name")]`** — Group properties under a toggleable inspector tab. `FeatureEnabled` paired with a bool turns the whole tab on or off.

**`FieldOfView`** — `CameraComponent.FieldOfView`, in degrees. Default 60. Higher values are more peripheral; lower values produce a "zoom" feel.

**`[FilePath]`** — Property attribute on a string that opens a file picker in the inspector instead of a text field.

**`FileSystem`** — The virtual filesystem. `FileSystem.Mounted` for read-only mounted addon content (`models/`, `materials/`, etc.). `FileSystem.Data` for user-writable data (saves, settings).

**`FindMode`** — Enum filter for component lookup. `EnabledInSelf`, `EverythingInSelfAndDescendants`, etc. Passed to `Components.Get<T>(FindMode)` and friends.

**`[FGDType("type")]`** — Attribute on a `[Property]` of a `HammerEntityDefinition` that hints which Hammer editor widget to use (`"team"`, `"studio"`, `"target_destination"`, etc.).

**Forge** — The THEMIS orchestrator that runs the wiki-generation agents (you, in earlier sessions). Each commit on `main` is one Forge iteration.

## G

**`GameObject`** — A thing in the scene. Holds a transform, tags, components, and children. Components do the work; the GameObject is the container. See [GameObject](scene/gameobject.md).

**`GameObject.Tags`** — A set of strings used for filtering (physics collision, render groups, custom queries). Inherited from parent to child by default.

**`GameObject.Network`** — The networking surface on a GameObject — `NetworkSpawn`, `IsOwner`, `Refresh`, `SetOwnerTransfer`, `Interpolation`. See [Networked Objects](systems/networking-multiplayer/networked-objects.md).

**`GameObject.IsValid()`** — Returns false if the GameObject has been destroyed. Always check before using a stored reference.

**`GameObjectSystem`** — A scene-level system that runs at specific frame phases. Use it instead of a `Component` when you need ordered cross-cutting work (gameplay-system tick that runs before all components, etc.). See [GameObjectSystem](scene/gameobjectsystem.md).

**`GameResource`** — Custom asset type defined by inheriting from `GameResource`. Inspector-edited, JSON-serialised, mountable from any addon. See [GameResource Extensions](systems/assetsresources/gameresource-extensions.md).

**`GameTask`** — `Sandbox.GameTask` — async helpers that integrate with the engine's tick (`GameTask.RunInThreadAsync`, `GameTask.DelayRealtimeSeconds(...)`). Prefer over raw `Task` when you need engine-aware scheduling.

**`Gizmo.Draw`** — Editor-time drawing API. Use in `OnUpdate` or `DrawGizmos` overrides for selection handles, debug lines visible in the scene editor.

**`[Group("name")]`** — Property attribute that puts the property in a collapsible group in the inspector.

## H

**Hammer** — The legacy `.vmap`-based level editor inherited from Source 2. Being deprecated in favour of Scene Mapping. See [Hammer (Mapping)](editor/hammer.md).

**`[HammerEntity]`** — Marks a class derived from `HammerEntityDefinition` as a placeable entity in Hammer. Pairs with `[Library("classname")]`.

**`HammerMesh`** — Component (different from `MeshComponent`) for mesh geometry tied to entities in a Hammer-authored map. Bridge between Hammer brushwork and the scene system. See [Hammer Mesh](scene/components/reference/hammer-mesh.md).

**`HelpUrl`** — Class attribute pointing to documentation. Hovering the inspector header shows a "?" link.

**`[Hide]`** — Class or property attribute that hides it from the inspector. Used for components like `MeshComponent` that are added by tools, not manually.

**`HighlightOutline`** — Component that adds a selection-style outline render to the GameObject's model. See [Highlight Outline](scene/components/reference/highlightoutline.md).

**Host** — The peer in a multiplayer session that owns the canonical world state. In P2P, the host is the player who created the lobby. `Networking.IsHost` checks this.

**Hotload** — The engine's recompile-and-replace cycle: save a `.cs` file → `Sandbox.Compiling` produces a new assembly → `Sandbox.Hotload` walks the heap and migrates live state to the new types. See [Hotloading](code/code-basics/hotloading.md). The `IL Hotload` fast path patches method bodies in place when only bodies changed.

**`HudPainter`** — Immediate-mode 2D drawing API for HUDs. Implement `IHudPainter`/`HudPainter` to draw shapes, text, lines per-frame. See [HudPainter](systems/ui/hudpainter.md).

## I

**`[Icon("material_icon_name")]`** — Class or attribute that gives a Material-icon name for the inspector header / scene-tree icon.

**IL Hotload** — The fast-path hotload that swaps method bodies without walking the heap. Almost instant. Used when only method bodies (not type definitions) changed.

**`IndirectLightVolume`** — Component for baked indirect lighting volumes. Used to add bounce-light to large rooms without expensive runtime GI.

**Inspector** — The editor panel that shows a GameObject's components and their `[Property]`-marked fields. Configurable via attributes like `[Range]`, `[Title]`, `[Group]`.

**Interceptor** — In `Sandbox.Mounting`, a component that handles a specific file format during mount (e.g., GoldSrc `.mdl` interceptor).

**InteropGen** — The tool at `engine/Tools/InteropGen` that parses C++ headers and generates `Sandbox.Bind` P/Invoke wrappers. You only run this if you're contributing to the engine and adding native bindings.

**`IsProxy`** — `true` when this client is *not* the owner of the GameObject. The most-common networking guard: `if ( IsProxy ) return;` to prevent non-owners from running owner-only code. See [Ownership](systems/networking-multiplayer/ownership.md).

**`IScenePhysicsEvents`** — Scene event interface for hooking into the physics step. `PrePhysicsStep` / `PostPhysicsStep`. Implement on a `Component` or `GameObjectSystem`. See [Physics Events](systems/physics/physics-events.md).

**`ISceneStartup`** — Scene event interface called when a scene first becomes active. `OnHostInitialize`, `OnClientInitialize`. See [ISceneStartup](scene/components/events/iscenestartup.md).

**`IGameObjectNetworkEvents`** — Scene event interface for per-GameObject network events: `NetworkOwnerChanged`, `StartControl`, `StopControl`. See [IGameObjectNetworkEvents](scene/components/events/igameobjectnetworkevents.md).

**`IsTrigger`** — `Collider.IsTrigger` — when true, the collider doesn't physically block but still detects entry/exit. Pair with `Component.ITriggerListener` to receive events.

## J

**Joints** — Components connecting two `Rigidbody`s (`HingeJoint`, `SliderJoint`, `BallJoint`, `FixedJoint`, `SpringJoint`, `UprightJoint`). See [Joints](systems/physics/joints.md).

**`Jump(force)`** — `PlayerController.Jump(Vector3 force)` — apply a vertical velocity burst respecting the controller's grounded state. Use this for double-jumps from extending PlayerController.

## K

**Kinematic** — A `Rigidbody` whose position is set by code rather than simulated by physics. Other dynamics react to it; it doesn't react to them. Set via `Rigidbody.MotionEnabled = false` (or "Kinematic" mode in the inspector).

**`[KeyProperty]`** — Marks a property of a custom Class/Struct as the "key" property — represented inline in the inspector with an "expand" affordance for the rest. Used in inspector UX for compact custom types.

## L

**`LaunchArguments`** — Static class capturing arguments the engine was launched with: `LaunchArguments.Map`, `MaxPlayers`, `Privacy`. `MapInstance.UseMapFromLaunch` reads `LaunchArguments.Map` to pick the boot map.

**Level** — Not an s&box term. A loadable, packaged level is called a **[Map](#m)**; the runtime tree of a loaded level is called a **[Scene](#s)**. See [Maps](scene/maps/index.md) for the runtime entry point and [Scene Mapping](scene/scene-mapping/index.md) for authoring.

**`[Library("classname")]`** — Stable identifier for a class. Required for `[HammerEntity]` definitions. Renaming the library name breaks existing maps that reference it.

**`LipSync`** — Component (file: `LipSyncComponent.cs`) that drives morph-target weights on a model from the spectrum of a sound playing nearby.

**Lobby** — A multiplayer session created via `Networking.CreateLobby(...)`. Has `MaxPlayers`, `Privacy` (`Public`/`FriendsOnly`/`Private`), and a name. See [Networking & Multiplayer](systems/networking-multiplayer/index.md).

**`LobbyConfig`** — The struct passed to `Networking.CreateLobby`. Sets max players, privacy, name, and other lobby parameters.

## M

**Mapping (tool)** — The in-scene mesh-editing toolset (`MeshTool`) for authoring level geometry. Subtools include Block, Face, Edge, Vertex, Texture, Vertex Paint, Displacement, plus modifiers (Bevel, Bridge, Clip, EdgeArch, EdgeCut, Mirror). Replaces Hammer brushwork. See [Scene Mapping](scene/scene-mapping/index.md).

**Map** — A loadable level. Either a Hammer-compiled `.vpk`, a scene-mapped `.scene`, or a package ident pointing to either. See [Maps](scene/maps/index.md).

**`MapInstance`** — Runtime component that loads and manages a map. Set `MapName` and the engine async-loads it. See [Map Instance](scene/components/reference/mapinstance.md) and [Loading Maps](scene/maps/loading-maps.md).

**Map Entity** — A placeable thing inside a Hammer map. Defined by a `[HammerEntity]` class deriving from `HammerEntityDefinition`. At runtime, each becomes a child `GameObject` of `MapInstance` with a `MapObjectComponent`.

**`MapObjectComponent`** — Internal component attached to map-entity GameObjects at load time, holding their `SceneObject`s and inheriting their tags from Hammer. You don't add it manually. See [Map Entities](scene/maps/map-entities.md).

**Material** — `Material` resource (`.vmat`) — describes how a surface is rendered (textures, shader, parameters). `Model.Materials[i]` for swapping per-mesh.

**`MathX`** — Static math helpers: `MathX.Lerp`, `MathX.Remap`, `MathX.Clamp`, `MathX.SmoothStep`. Beats writing them yourself.

**`MeshComponent`** — Runtime component that owns a `PolygonMesh`, producing a render model and a collider. Added by the Mapping tool's primitive editors. `[Hide]`-marked. See [Mesh Component](scene/components/reference/meshcomponent.md).

**`MeshTool`** — `[Title("Mapping")]` — the engine's in-scene mesh editor. Top-level toolbar entry. See [Scene Mapping](scene/scene-mapping/index.md).

**Mixer** — Audio bus (Music / SFX / Voice / UI) that groups sounds for routing and global volume control. `Mixer.FindMixerByName(name).Volume = 0.5f`.

**`Model.Load(path)`** — Static loader that returns a `Model` from a `.vmdl` path. Models are cached internally.

**`ModelPhysics`** — Component that builds a multi-body physics representation from a model's joints. Used for ragdolls. See [Model Physics](scene/components/reference/modelphysics.md).

**`ModelRenderer`** — Component that draws a `Model` at the GameObject's transform. Properties: `Model`, `Tint`, `MaterialOverride`, render flags.

**Mount** — Both a noun and a verb. As a noun, an external game (GoldSrc, NS2, Quake) whose assets s&box can read. As a verb, the act of attaching a directory to the virtual filesystem so its files become reachable. See [Mounting](systems/mounting/index.md).

**`MusicPlayer`** — Streaming-audio player (separate from sound effects). `MusicPlayer.Play(fs, path)`, `MusicPlayer.PlayUrl(url)`. Exposes `Volume`, `Repeat`, `Spectrum`, `Amplitude`. See [Audio (Streaming)](systems/media/audio.md).

## N

**NavMesh** — The runtime mesh used for AI pathfinding. Generated from level collision. `Scene.NavMesh.GetClosestPoint(...)`, `Scene.NavMesh.GetRandomPoint(...)`.

**`NavMeshAgent`** — Component for an AI agent moving on the NavMesh. `MoveTo(pos)`, `Stop()`, `IsNavigating`. See [NavMesh Agent](scene/components/reference/navmeshagent.md).

**`NetDictionary<K,V>`** — Networked dictionary that replicates inserts/removes individually. Use instead of `Dictionary<K,V>` for `[Sync]`-marked maps.

**`NetList<T>`** — Networked list. Same idea — use instead of `List<T>` for `[Sync]`-marked collections.

**`NetworkSpawn()`** — Method on `GameObject` that registers it with the network so all peers receive it. `NetworkSpawn(connection)` assigns ownership to a specific client.

**Network Helper** — `NetworkHelper` component in the `base` addon. Drop-in lobby + auto-spawning helper for fast multiplayer prototypes. See [Network Helper](systems/networking-multiplayer/network-helper.md).

**`NetworkMode`** — Enum on `GameObject.NetworkMode`. `Object` = full network sync, `Snapshot` = part of the initial scene snapshot but not actively replicated, `Never` = local-only.

**`Networking`** — Static class. `Networking.IsActive`, `Networking.IsHost`, `Networking.IsClient`, `Networking.CreateLobby(...)`, `Networking.Connect(...)`.

## O

**`OnAwake`** — First lifecycle method after a component is created. Other components on the same GameObject may not have awoken yet; don't query siblings here. Override to do internal-only setup.

**`OnDestroy`** — Final lifecycle method. The component or GameObject is being removed. Release external resources here.

**`OnEnabled` / `OnDisabled`** — Fire whenever the component flips between enabled/disabled. Use to subscribe/unsubscribe from external events.

**`OnFixedUpdate`** — Lifecycle method that runs at a fixed timestep (~50 Hz). For physics-driving code; consistent across framerates.

**`OnHostInitialize` / `OnClientInitialize`** — Methods on `ISceneStartup`. Called once when the scene becomes active on host or client respectively.

**`OnLoad()` / `OnLoad(LoadingContext)`** — Async lifecycle method. The loading screen waits for the returned `Task`. Used for async data setup before the scene starts.

**`OnPreRender`** — Called every frame just before rendering. Useful for visual adjustments after animation has computed bone positions but before the render queue is built. Never called on dedicated servers.

**`OnRefresh`** — Called after hotload or deserialization. Use to rebuild caches, re-subscribe to delegates that hotload couldn't preserve.

**`OnStart`** — Lifecycle method called once when the component becomes fully active, after every component on the GameObject has finished `OnAwake`. Safe to query siblings here.

**`OnUpdate`** — Lifecycle method called every rendered frame. The game-logic hot path. Multiply continuous changes by `Time.Delta`.

**`OnValidate`** — Called when a property is changed in the inspector or after deserialization. Override to clamp values or enforce invariants.

**`[Order(int)]`** — Property attribute that sets the inspector display order. Lower numbers appear first.

**Owner** — The peer with authority over a networked GameObject. The owner's `[Sync]` writes are authoritative; non-owners see the snapshot. `Connection` reference exposed via `GameObject.Network.Owner`.

**OwnerTransfer** — Enum on `GameObject.Network.SetOwnerTransfer(...)`. `Fixed` (host-only), `Takeover` (any client), `Request` (request-from-host).

## P

**Package** — A bundled addon, game project, or standalone export, identified by an ident like `facepunch.walker`. Discovered through `Package.Fetch(ident)`.

**`PanelComponent`** — Razor-driven UI component. Pairs with a `.razor` file describing the panel markup. Add `ScreenPanel` or `WorldPanel` to the same GameObject to decide where it renders.

**Physics World** — The collision/simulation context for a `Scene`. `Scene.PhysicsWorld`. Each scene has its own.

**`PhysicsType`** — `Rigidbody.PhysicsType` — `Static`, `Kinematic`, or `Dynamic`. Decides whether physics simulates the body, you set its position by code, or it's immovable.

**`PolygonMesh`** — Editable half-edge mesh data type owned by `MeshComponent`. The thing the Mapping tool actually edits. See [Polygon Mesh](scene/components/reference/polygonmesh.md).

**Prefab** — A saved GameObject hierarchy that can be instantiated multiple times. `.prefab` file. Spawn via `prefab.Clone()` or `Scene.PrefabSystem.Spawn(...)`. See [Prefabs](scene/prefabs/index.md).

**`[Property]`** — Attribute that exposes a property to the inspector and to ActionGraph. Pairs with other attributes (`[Range]`, `[Title]`, `[Group]`, etc.) for richer editor controls.

**Proxy** — In networking, "the local peer is not the owner of this object." See `IsProxy`.

**Pulse / Data nodes** — In ActionGraph, pulse nodes are execution-flow nodes (like `OnStart`, `If`); data nodes evaluate values when needed (like `GameObject.WorldPosition`).

## Q

**Qt** — The C++ UI framework underlying the s&box editor. `Sandbox.Tools/Qt/` wraps it for C# editor extensions. You don't touch Qt directly when writing games — only when extending the editor.

## R

**`[Range(min, max, slider: bool)]`** — Property attribute that constrains a numeric value. With `slider: true`, renders a slider in the inspector.

**Razor** — The HTML-like markup language used for UI. `.razor` files compile via `Sandbox.Razor` into C# panel classes. Bind C# values into markup with `@property`. See [Razor Panels](systems/ui/razor-panels/index.md).

**`Rigidbody`** — Physics-simulating component. Has mass, gravity, velocity. Pairs with a `Collider`. Use `ApplyImpulse(force)` for instant pushes, `ApplyForce(force)` for continuous force. See [Rigidbody](systems/physics/rigidbody.md).

**Roslyn** — The .NET compiler platform that `Sandbox.Compiling` uses to compile your `.cs` files to a hot-loadable assembly.

**`Rotation`** — Quaternion-backed math type. Compose rotations with multiplication: `a * b`. Build from common forms: `Rotation.From(pitch, yaw, roll)`, `Rotation.FromAxis(axis, angle)`, `Rotation.LookAt(direction)`.

**`Rpc.Caller`** — Inside an RPC method body, the `Connection` that invoked the RPC.

**`[Rpc.Broadcast]`** — Attribute on a method making it run on every peer when called.

**`[Rpc.Host]`** — RPC variant that only runs on the host.

**`[Rpc.Owner]`** — RPC variant that only runs on the owner of the GameObject the component is attached to.

**Rubikon** — Source 2's underlying physics engine. You don't see it directly; the C# `Rigidbody`/`Collider` API drives it.

## S

**Script** — Not an s&box term. There's no separate "script" type — every piece of game logic is a **[Component](#c)** class derived from `Sandbox.Component`. See [Code](code/index.md).

**`[Sync]`** — Attribute that replicates a property's value across the network. Only the owner may write. Pair with `[Change(nameof(method))]` to react to changes. See [Sync Properties](systems/networking-multiplayer/sync-properties.md).

**`SyncFlags`** — Flags enum on `[Sync(SyncFlags.X)]`. `Query` (re-check getter every tick), `FromHost` (only host writes regardless of ownership), `Interpolate` (smoothly tween values for non-owners).

**Sandbox** — Two unrelated meanings:
1. The `Sandbox` namespace where the engine API lives.
2. The "sandbox" is the API whitelist that restricts what user code can call (security boundary).

**`Sandbox.AppSystem`** — See `AppSystem`.

**`Sandbox.Bind`** — Native interop project. Two roles: P/Invoke wrappers generated by `InteropGen`, and the property-binding system used by Razor UI (`Sandbox.Bind.Proxy`, `Sandbox.Bind.Links`).

**`Sandbox.Compiling`** — Runtime Roslyn compiler that turns your `.cs` files into a hot-loadable assembly. See [Sandbox.Compiling](engine-internals/sandbox-compiling.md).

**`Sandbox.Hotload`** — The deep-swap upgrader that walks the heap and migrates live state into a newly-compiled assembly. See [Sandbox.Hotload](engine-internals/sandbox-hotload.md).

**`Sandbox.Reflection`** — Engine reflection layer with the `TypeLibrary` queryable type/property metadata used by the inspector, ActionGraph, snapshots, and source generators.

**`.sbproj`** — Your project file. JSON metadata about your package: ident, dependencies, project type, mounted assets.

**Scene** — The runtime tree of GameObjects you're playing in. `Scene` is also the C# class — `Scene.Camera`, `Scene.Trace`, `Scene.GetAllComponents<T>()`, `Scene.NavMesh`. See [Scene System](scene/index.md).

**`.scene`** — A scene file. JSON, version-controlled, can be loaded as a Map via `MapInstance`.

**Scene Map** — A `.scene` file used as a map. Loaded via `MapInstance.MapName = "path/to.scene"`. The modern alternative to compiled `.vmap`/`.vpk` maps.

**Scene Mapping** — The in-scene level-authoring workflow. Use the Mapping tool to build geometry directly in the scene editor. See [Scene Mapping](scene/scene-mapping/index.md).

**`SceneTrace`** / **`SceneTraceResult`** — Builder and result types for ray/sphere/box/capsule/cylinder traces. `Scene.Trace.Ray(from, to).Run()` returns a `SceneTraceResult`. See [Trace / Raycast](systems/physics/physics-events.md).

**`SceneObject`** — Internal native render object. Components like `ModelRenderer` and `MeshComponent` produce these; you don't construct them directly except in advanced editor extensions.

**`[ShowIf("propertyName", value)]` / `[HideIf(...)]`** — Property attributes that conditionally show or hide a property based on another property's value.

**`SkinnedModelRenderer`** — Component for animated skeletal models (vs static `ModelRenderer`). Drives bones, exposes `AnimationGraph`, `PlaybackRate`, `BoneMergeTarget`.

**`[SkipHotload]`** — Attribute on a class or field telling the hotload upgrader to leave it alone. Used for native pointers, expensive caches you'll rebuild via `OnRefresh`.

**Snapshot** — The serialized state of the world (including all `[Sync]` properties and GameObject transforms) sent from the host to clients multiple times per second. This ensures that late-joiners get the current state immediately without needing to receive a history of RPC events.

**Proxy** — A state where the local peer is *not* the owner of a networked object. Proxies do not run their own physics simulation; instead, they smoothly follow the data received in snapshots from the authoritative host. Use `if ( IsProxy ) return;` to guard logic that should only run on the owner.

**`SoundEvent`** — Asset (`.sound`) that wraps audio files with playback rules: pitch randomization, attenuation, mixer routing. You play these via `Sound.Play(soundEvent, position)`.

**`SoundPointComponent`** / **`SoundBoxComponent`** — Components for continuous audio attached to a moving GameObject. `SoundPoint` plays from a single point; `SoundBox` plays from anywhere inside a volume.

**`SoundscapeTrigger`** — Component that switches the active soundscape (background ambience) when the listener enters its volume.

**Source 2** — The native C++ engine s&box runs on. Provides rendering (Vulkan/DX), physics (Rubikon), audio, networking transport. You don't write Source 2 code directly; `Sandbox.Bind` exposes the bits you need.

**Standalone** — A self-contained game export with its own `.exe`. Lets you publish to non-s&box platforms (Steam, itch). See [Exporting Standalone](publishing/exporting-standalone.md).

**Steam P2P** — The transport that powers casual multiplayer. The host is a player; clients connect through Steam's NAT-traversed P2P relay.

## T

**Tags** — String labels on GameObjects (`GameObject.Tags`). Inherited by children. Used for physics-collision filtering, render-group filtering, and ad-hoc query categorization.

**`Terrain`** — Component for large-scale heightmap-based terrain. See [Terrain Component](scene/components/reference/terraincomponent.md).

**`Time.Delta`** — Seconds elapsed since the previous `OnUpdate`. Always multiply continuous changes by it for framerate independence.

**`Time.FixedDelta`** — Time per fixed-update tick (typically `1/50 = 0.02`).

**`Time.Now`** — Current scene time in seconds. Stops when the scene is paused.

**`Time.TimeScale`** — `Scene.TimeScale` — global pause/slow-mo. `0` pauses; `0.5` is half-speed.

**`[Title("Display Name")]`** — Attribute that overrides the inspector display name for a property or class.

**`[ToggleGroup("name")]`** — Property attribute paired with a bool to group properties under a checkbox-toggleable section.

**Trace** — Cast a ray, sphere, box, capsule, or cylinder through the physics world to find what's there. `Scene.Trace.Ray(...).Run()`. See [Trace / Raycast](systems/physics/physics-events.md).

**Trigger** — A `Collider` with `IsTrigger = true`. Doesn't block movement, but fires `OnTriggerEnter` / `OnTriggerExit` events on listeners.

**`TypeLibrary`** — The reflection cache used by the inspector, ActionGraph, snapshots, source generators. `TypeLibrary.GetType(t)`, `TypeLibrary.GetTypes<T>()`. Engine-internal but exposed for advanced uses.

## U

**Upgrader** — In `Sandbox.CodeUpgrader`, a Roslyn rewriter that automatically migrates user C# when the engine API changes. Custom upgraders live in `engine/Sandbox.CodeUpgrader/Upgraders/`.

**`UseMapFromLaunch`** — `MapInstance.UseMapFromLaunch` — when true and `LaunchArguments.Map` is non-empty, that wins over the default `MapName`. Lets a single boot scene serve any map the menu picks.

## V

**`Vector3`** — 3-component float vector. `Vector3.Up = (0, 0, 1)`, `Vector3.Forward = (1, 0, 0)`, `Vector3.Right = (0, 1, 0)`. (s&box uses Z-up by default.)

**`VerletRope`** — Component for verlet-integrated cable/rope simulation.

**`VideoPlayer`** — Streams a video file or URL, exposes the current frame as a `Texture`. Use as a panel background, material parameter, or scene texture. See [Video](systems/media/video.md).

**`VideoWriter`** — Encodes RGBA frames into a video file. Used internally by the screen recorder; available for tools that capture replays.

**Voice** — `VoiceComponent` (`Voice` class) — captures microphone input and replicates to other peers. Pair with `LipSync` for animated speaking models.

**`.vmap`** — Hammer's source map file. Compiled to `.vpk` for runtime loading. Legacy mapping path; prefer `.scene` for new work.

**`.vpk`** — Compiled map file. Loaded directly by `MapInstance` or referenced by package ident.

## W

**`WideMode`** — Property attribute that makes the inspector render the property's editor below its label (full width) instead of beside it. For long expressions or compound values.

**`WorldPosition` / `WorldRotation` / `WorldScale`** — A GameObject's transform in world space. Set directly: `GameObject.WorldPosition = ...`. The legacy `Transform.Position` form on `GameTransform` is `[Obsolete]`.

**`WorldPanel`** — A `PanelComponent` rendered in 3D space (signs, name tags, holograms) instead of flat over the camera. Pairs with `WorldInput` to handle clicks.

**World** — Not the standard s&box term. The runtime world is called a **[Scene](#s)**. The data on disk for a level is a **[Map](#m)** (`.scene` or `.vpk`).

---

## Vocabulary from other engines

If you searched for a word you'd use elsewhere and didn't find it, this is the translation table.

### Unity → s&box

| Unity term | s&box equivalent | Notes |
|---|---|---|
| GameObject | `GameObject` | Same name, same concept |
| MonoBehaviour | `Component` | Inherit from `Component`, override `OnUpdate` etc. |
| Update / FixedUpdate / LateUpdate | `OnUpdate` / `OnFixedUpdate` / `OnPreRender` | See [Component Lifecycle](scene/components/component-lifecycle.md) |
| Awake / Start / OnEnable / OnDisable | `OnAwake` / `OnStart` / `OnEnabled` / `OnDisabled` | Same semantics |
| OnDestroy | `OnDestroy` | Same |
| Prefab | `Prefab` | Same name |
| Scene | `Scene` (the runtime tree); a saved scene file is `.scene` | |
| Serializable / [SerializeField] | `[Property]` | Surfaces in the inspector |
| Transform | `Transform` (struct) and `GameTransform` (instance) — but use `WorldPosition` / `WorldRotation` / `WorldScale` directly on the GameObject | |
| RaycastHit / Physics.Raycast | `SceneTraceResult` / `Scene.Trace.Ray(...)` | See [Trace](systems/physics/physics-events.md) |
| AudioSource | `SoundPointComponent` (3D) or `Sound.Play()` (one-shot) | |
| AudioClip | `SoundEvent` (`.sound` asset) | Wraps the raw audio with playback rules |
| Rigidbody | `Rigidbody` | Same name |
| Collider | `Collider` (`BoxCollider`, `SphereCollider`, etc.) | Same idea |
| NavMeshAgent | `NavMeshAgent` | Same |
| Coroutine / IEnumerator | `async Task` | Use `async/await`; `OnLoad` can be async |
| ScriptableObject | `GameResource` | Custom asset type, JSON-serialised |
| Visual Scripting (Bolt) | `ActionGraph` | |
| Inspector | Inspector | Same name |
| Hierarchy | Scene Tree | |
| Resources.Load | `FileSystem.Mounted` | Or via `[Property]` references which are preferable |
| PlayerPrefs | `Cookies` (transient) or `FileSystem.Data` (persistent) | |

### Unreal → s&box

| Unreal term | s&box equivalent | Notes |
|---|---|---|
| Actor | `GameObject` | A GameObject is much lighter than an Actor — no class hierarchy to inherit from |
| ActorComponent | `Component` | Drop-in attached to a GameObject |
| Tick / TickComponent | `OnUpdate` / `OnFixedUpdate` | |
| BeginPlay / EndPlay | `OnStart` / `OnDestroy` | |
| Pawn / Character | No direct equivalent — use [`PlayerController`](scene/components/reference/player-controller.md) component on a GameObject | |
| GameMode / GameInstance | Just a `Component` on a designated GameObject | s&box has no game-mode class hierarchy |
| Blueprint | `ActionGraph` | Visual scripting; most s&box devs write C# directly |
| Blueprint Class (BP) | `Prefab` | Reusable saved hierarchy |
| Level | `Map` (the loadable asset) or `Scene` (the runtime tree) | See [Maps](scene/maps/index.md) |
| World | `Scene` | The runtime container |
| StaticMeshComponent | `ModelRenderer` | |
| SkeletalMeshComponent | `SkinnedModelRenderer` | |
| SoundCue | `SoundEvent` (`.sound`) | |
| AnimBlueprint | (No direct equivalent — Source 2 anim graphs handle most of this) | |
| UPROPERTY | `[Property]` | Editor exposure + serialization |
| UFUNCTION(BlueprintCallable) | Just a `public` method on a `Component` | ActionGraph can call any public method via reflection |
| Replicated UPROPERTY | `[Sync]` | See [Sync Properties](systems/networking-multiplayer/sync-properties.md) |
| Multicast UFUNCTION | `[Rpc.Broadcast]` | |
| Server / Client UFUNCTION | `[Rpc.Host]` / `[Rpc.Owner]` | |
| HasAuthority() | `!IsProxy` (you are the owner) or `Networking.IsHost` (you are the host) | See [Ownership](systems/networking-multiplayer/ownership.md) |
| FindActorsWithTag | `Scene.GetAllComponents<T>().Where(c => c.GameObject.Tags.Has("..."))` | |
| Cast<T> | Standard C# `as T` | s&box uses regular C# casts; no special macro |
| Slate / UMG | Razor panels | See [UI](systems/ui/index.md) |
| Hammer (Source 1, separate tool) | `Hammer` (legacy) — being deprecated. Use [Scene Mapping](scene/scene-mapping/index.md) for new work | |
| GAMEMODE / hook.Add (GMod) | `Component`s + `[Event]` attributes / `IScene...` event interfaces | No hook system; use C# events directly |

### Generic / "I just say it like a regular dev" → s&box

| You said | s&box term |
|---|---|
| "Script" | `Component` (s&box has no separate script type) |
| "Level" | `Map` (loadable asset) or `Scene` (runtime tree) |
| "World" | `Scene` |
| "Entity" | `GameObject` (with one or more `Component`s) |
| "Behaviour" | `Component` |
| "Actor" | `GameObject` |
| "Player object" | `GameObject` with a [`PlayerController`](scene/components/reference/player-controller.md) component |
| "Particle effect" | [`ParticleEffect`](systems/effects/particle-effect/index.md) component |
| "Decal" | [`Decal`](scene/components/reference/decal.md) component |
| "Cinematic" | Movie Maker — see [Movie Maker](systems/movie-maker/index.md) |
| "Hot reload" / "live reload" | Hotload — see [Hotloading](code/code-basics/hotloading.md) |
| "Save game" | `FileSystem.Data` for files; `Cookies` for ephemeral key/value | |
| "Build pipeline" | `engine/Tools/SboxBuild` and `Bootstrap.bat` |
| "Native plugin" | `Sandbox.Bind` (P/Invoke wrappers; not user-extensible) |
| "DLC" | A separately-published Package mounted at runtime |
| "Workshop / community content" | sbox.game packages — discovered and downloaded automatically |
| "Async / coroutine" | `async Task` — see [Use Async Tasks](how-to/component-async.md) |
| "Singleton manager" | A `Component` on a designated GameObject, or a `GameObjectSystem` |

## Looking for something not here?

The wiki itself is the deepest reference — the [Build Games](build-games/index.md) track is probably what you want, or [Engine Internals](engine-internals/index.md) if you're working on the runtime. For symptom-driven debugging see [Troubleshooting](troubleshooting/index.md).
