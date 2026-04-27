---
title: "Engine Systems"
icon: "🏭"
sources:
  - engine/Sandbox.Engine/Systems/
updated: 2026-04-25
created: 2026-04-27
---

# Engine Systems

The engine is split into systems by domain — physics, networking, audio, UI, navigation, and so on. Each system has a few `Component` types you'll attach to GameObjects (like `Rigidbody`, `CameraComponent`, `NavMeshAgent`) plus some global accessors on `Scene` for things that don't make sense as a single component (like `Scene.Trace.Ray(...)`).

This section is the reference for those systems. Each subsection has its own landing page and goes from there.

## Common shapes

Most systems expose two surfaces:

- **Components you place** — `Rigidbody`, `CameraComponent`, `SoundPointComponent`, `NavMeshAgent`. You add these to GameObjects and configure them in the Inspector.
- **Static accessors on `Scene`** — `Scene.Trace.Ray(...)`, `Scene.NavMesh.GetClosestPoint(...)`, `Scene.GetAllComponents<T>()`. These query the active scene without needing a component instance.

If you can't find a feature exposed as a component, look for a `Scene.X` accessor. If neither, check the system's landing page for the underlying type.

## Sections

### Core gameplay
- [Physics](physics/index.md) — `Rigidbody`, `Collider`, `Joint`, `Scene.Trace`, triggers, physics events.
- [Input](input/index.md) — keyboard, mouse, controller, VR, raw input.
- [Navigation](navigation/index.md) — NavMesh generation, `NavMeshAgent`, areas, links.
- [Performance](performance/index.md) — profiling, optimization patterns.

### Multiplayer & online
- [Networking & Multiplayer](networking-multiplayer/index.md) — `[Sync]`, RPCs, ownership, dedicated servers.
- [Services](services/index.md) — leaderboards, achievements, stats, auth.
- [Platform](platform/index.md) — Steam, VR, streaming integrations, standalone export.

### Visuals
- [Shaders](shaders/index.md) — HLSL shaders, render attributes, materials.
- [Shader Graph](shader-graph/index.md) — node-based shader authoring.
- [Effects](effects/index.md) — particles, decals, beams, light volumes.
- [Post-Processing](post-processing/index.md) — full-screen effects.
- [UI](ui/index.md) — Razor panels, screen/world panels, HUD painter.
- [Media](media/index.md) — `MusicPlayer` for streaming audio, `VideoPlayer` for video textures.

### Content & assets
- [Audio (gameplay)](audio/index.md) — sound components, mixers, DSP, voice chat.
- [Assets & Resources](assetsresources/index.md) — `GameResource`, custom asset types, cloud assets.
- [File System](file-system/index.md) — VFS, mounted addons, user-data storage.
- [Mounting](mounting/index.md) — bringing in external games (GoldSrc, NS2, Quake) and writing your own mounts.

### Authoring
- [Movie Maker](movie-maker/index.md) — cinematics, recording, animation editing.
- [ActionGraph](actiongraph/index.md) — visual scripting.
- [Scene-side editor tools](../editor/index.md) — Mapping (`MeshTool`), Hammer, Shader Graph, Movie Maker apps.

### Misc
- [VR](vr/index.md) — anchors, hands, tracked objects.
- [Avatar](avatar/index.md) — Citizen avatar customization.
- [Clutter](clutter/index.md), [Terrain](terrain/index.md) — procedural and large-scale world systems.
- [Security](security/index.md) — sandbox boundaries.

## Things that surprise people

**Most queries scope to a Scene.** There's no global `Physics.Trace` static — it's `Scene.Trace`. If you have multiple scenes loaded, each has its own physics world, navmesh, etc. Keep that in mind for editor tools that operate across scenes.

**A system without its required component does nothing.** Audio with no `AudioListener`. Physics with no `Rigidbody` + `Collider` pair. Rendering with no `CameraComponent`. The default scene template has all of these; if you build a scene from scratch in code, add them.

**Initialization timing.** Querying `NavMesh` or trying to spawn networked objects before the scene has fully loaded gets you empty/invalid data. Wait for `OnStart` (or for `INetworkListener.OnActive` for networking).

## Read next

If you're new and just want to see things working, [the Sweeper sample](../build-games/samples-templates/sweeper.md) uses physics, UI, and gameplay components in a small complete project — handy as a reading exercise.

For deep architectural reading: [Architectural Overview](../architecture.md) and [Engine Internals](../engine-internals/index.md).
