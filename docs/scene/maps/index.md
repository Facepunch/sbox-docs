---
title: "Maps"
icon: "🌍"
created: 2026-04-25
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Game/Map/Map.cs
  - engine/Sandbox.Engine/Game/Map/MapLoader.cs
  - engine/Sandbox.Engine/Scene/Components/Map/MapInstance.cs
  - engine/Sandbox.Engine/Scene/Components/Map/MapObjectComponent.cs
---

# Maps

A "map" in s&box is a packaged level — a saved scene (`.scene`), Hammer geometry (`.vmap` → compiled `.vpk`), or a Steam Workshop / asset.party package — that you load into a running scene at runtime. Maps are first-class but distinct from scenes: a `Scene` is the runtime world, a Map is content that gets *loaded into* a scene.

The runtime entry point is the `MapInstance` component. Drop one onto a GameObject, set `MapName`, and the map loads asynchronously — its geometry, lights, and entities are spawned as children of the `MapInstance`'s GameObject.

> **Authoring vs. loading.** This section covers **loading** maps at runtime. To **author** a new map, use [Scene Mapping](../scene-mapping/index.md) — the in-scene Mapping tool that produces `.scene` files. Hammer (`.vmap`) is the legacy alternative; see [Hammer (Mapping)](../../editor/hammer.md) for the deprecation context.

## Subpages

- [**Loading Maps**](loading-maps.md) — `MapInstance.MapName`, `UseMapFromLaunch`, `OnMapLoaded`/`OnMapUnloaded`, `IsLoaded`, `Bounds`, `UnloadMap`. How to load by package ident, `.vmap`, or `.vpk`.
- [**Map Entities**](map-entities.md) — Hammer entities at runtime. `[HammerEntity]`, `HammerEntityDefinition`, `[Library]` for class names, `[FGDType]`, custom point/solid/path/cable classes, `MapObjectComponent`.
- [**Map Networking**](map-networking.md) — How `MapInstance` interacts with the snapshot system, server-driven map changes, and the launch-argument flow when a client connects.

## Quick Working Example

```csharp
using Sandbox;

public sealed class LevelLoader : Component
{
    [Property, MapAssetPath] public string MapName { get; set; } = "facepunch.construct";

    private MapInstance map;

    protected override void OnStart()
    {
        // Spawn a child holding the MapInstance
        var go = new GameObject( true, "Map" );
        go.SetParent( GameObject );

        map = go.Components.Create<MapInstance>();
        map.OnMapLoaded += () => Log.Info( $"Loaded {map.MapName} (bounds {map.Bounds})" );
        map.OnMapUnloaded += () => Log.Info( "Map unloaded" );

        // Setting MapName triggers an async load
        map.MapName = MapName;
    }
}
```

## What lives where

| Concern | Source | Notes |
|---|---|---|
| Component you place in scenes | `MapInstance.cs` | The runtime owner of one loaded map |
| Async load orchestration | `MapInstance.LoadMapAsync` | Cancellable, semaphore-guarded |
| Geometry + physics | `Map.cs`, `MapLoader.cs` | Wraps `SceneMap` and `PhysicsGroup` |
| Per-entity child GameObject | `MapObjectComponent.cs` | Holds the `SceneObject` list, copies tags from Hammer |
| Entity definitions (Hammer) | `engine/Sandbox.Tools/MapEditor/HammerEntities/` | Editor-only, `[HammerEntity]`-attributed |
| FGD / class metadata | `engine/Sandbox.Tools/GameData/MapClass.cs` | Editor reflection of map classes |
| Launch-time map selection | `engine/Sandbox.Engine/Game/LaunchArguments.cs` | `LaunchArguments.Map`, used when `UseMapFromLaunch` is true |

## Key Properties at a Glance

| Property | Type | Description |
|---|---|---|
| `MapName` | `string` | Map ident or path. Setting this triggers an async load on next `OnUpdate`. |
| `UseMapFromLaunch` | `bool` | If true and `LaunchArguments.Map` is set, that wins on first load. |
| `EnableCollision` | `bool` | Whether the loaded map's physics geometry is active. |
| `IsLoaded` | `bool` (read-only) | True when the underlying `SceneMap` is non-null. |
| `Bounds` | `BBox` (read-only) | World-space bounds of the loaded map. Default `BBox` if not loaded. |
| `OnMapLoaded` | `Action` | Invoked after the map finishes loading. |
| `OnMapUnloaded` | `Action` | Invoked after the map is torn down. |

- **Map name changes during load.** Setting `MapName` while a load is in flight cancels the in-flight load and starts a new one. The `OnMapLoaded` event only fires for the *final* successful load.
- **Editor mode.** In the editor, child GameObjects that were saved into the scene file are *not* deleted on unload — only objects flagged `GameObjectFlags.NotSaved` are reaped. Runtime children spawned by the map system are flagged `NotSaved` automatically.
- **Reflections.** Loading or unloading a map dirties the world reflection probes; `MapInstance` automatically subscribes to its own `OnMapLoaded` / `OnMapUnloaded` to call `UpdateDirtyReflections`.
- **Disabled component.** Disabling a `MapInstance` calls `UnloadMap()`. Re-enabling it re-loads from `MapName`.

## Related Pages

- [Hammer (mapping)](../../editor/hammer.md) — authoring the maps
- [Map Instance component reference](../components/reference/mapinstance.md)
- [External Game Mounting](../../systems/mounting/index.md) — when maps come from mounted games
