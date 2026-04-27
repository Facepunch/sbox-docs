---
title: "Loading Maps"
icon: "↗️"
created: 2026-04-25
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Scene/Components/Map/MapInstance.cs
  - engine/Sandbox.Engine/Game/LaunchArguments.cs
  - engine/Sandbox.Engine/Game/Map/Map.cs
---

# Loading Maps

> **New to maps?** The Maps overview is at **[Maps](index.md)**. For authoring (the **Mapping** tool), see **[Scene Mapping](../scene-mapping/index.md)**. This page is a recipe for one specific task: loading a map at runtime.

`MapInstance` is the only runtime entry point you need to load a map. It accepts four shapes of `MapName`:

| Shape | Example | Behavior |
|---|---|---|
| Package ident | `"facepunch.construct"` | Looked up via `Package.Fetch`. Downloaded if not already mounted. Must have `TypeName == "map"`. |
| `.vpk` path | `"maps/dm_lobby.vpk"` | Loaded directly from the mounted filesystem. |
| `.vmap` path | `"maps/dm_lobby.vmap"` | Auto-converted to the matching `.vpk` before loading. |
| `.scene` path | `"levels/arena.scene"` | **Scene Map.** Loads a `.scene` asset authored with [Scene Mapping](../scene-mapping/index.md). The scene's GameObjects are spawned as children of the `MapInstance` just like Hammer entities. |

> **Scene Mapping is the recommended path for new projects.** Build the level inside the scene editor with the **Mapping** tool — see [Scene Mapping](../scene-mapping/index.md) for the full workflow. Then point `MapInstance.MapName` at the `.scene` file. See [Hammer (Mapping)](../../editor/hammer.md) for the legacy `.vmap` alternative.

## Triggering a Load

Setting `MapName` does **not** load synchronously. `MapInstance.OnUpdate` watches for `loadedMapName != MapName` and kicks off `LoadMapAsync` on the next tick:

```csharp
map.MapName = "facepunch.construct";
// Loading is in flight; map.IsLoaded is still false until it completes.
```

If you need to know when the load actually finishes, subscribe to `OnMapLoaded`:

```csharp
map.OnMapLoaded += () =>
{
    Log.Info( $"{map.MapName} ready, bounds = {map.Bounds}" );
    SpawnPlayer();
};
```

The first frame `OnUpdate` runs after a `MapName` change is when the load is queued, so `OnMapLoaded` typically fires within one or two frames for a small map plus the time to download / parse for larger maps.

## Loading from Launch Arguments

When the engine launches your game with a map argument (e.g. from the menu's "Play" button), `LaunchArguments.Map` is populated. To respect that, set `UseMapFromLaunch = true`:

```csharp
[Property, MapAssetPath] public string DefaultMap { get; set; } = "facepunch.construct";

protected override void OnStart()
{
    var map = Components.Create<MapInstance>();
    map.UseMapFromLaunch = true;
    map.MapName = DefaultMap; // fallback used only if LaunchArguments.Map is empty
}
```

When `UseMapFromLaunch` is true and `LaunchArguments.Map` is non-empty, `MapInstance` overwrites `MapName` with the launch value before loading. This lets a single scene serve as the boot scene for any map the player picks from the menu.

## Inspecting Loaded State

```csharp
if ( map.IsLoaded )
{
    var bounds = map.Bounds;     // BBox in world space
    Log.Info( $"Map is loaded, world bounds: {bounds}" );
}
else
{
    Log.Info( "Still loading or no map set" );
}
```

`IsLoaded` is true exactly when the internal `SceneMap` reference is alive. `Bounds` returns `default` (zero-extent `BBox`) when no map is loaded.

## Unloading

Two ways to unload:

1. **Disable the component.** `MapInstance.OnDisabledInternal` calls `UnloadMap` automatically.
2. **Call `UnloadMap()` directly.** Tears down geometry, physics, and child GameObjects, then fires `OnMapUnloaded`.

```csharp
map.UnloadMap();
// map.IsLoaded is now false; map.MapName is unchanged.
```

Setting `MapName` to a new value also implicitly unloads the previous map before loading the new one.

## Switching Maps

To swap maps, just assign a new `MapName`:

```csharp
map.MapName = "facepunch.flatgrass";
```

Internally this:

1. Cancels any in-flight load via the per-instance cancellation token set.
2. Calls `UnloadMap()` to tear down the previous map.
3. Awaits the load semaphore (only one load at a time per `MapInstance`).
4. Loads the new map.
5. Fires `OnMapLoaded`.

If you assign `MapName` rapidly, intermediate values are skipped — `OnMapLoaded` only fires for whichever value is current when the semaphore is acquired.

## Common Patterns

### Wait for the map before spawning

```csharp
public sealed class GameManager : Component
{
    [Property] public MapInstance Map { get; set; }
    [Property] public GameObject PlayerPrefab { get; set; }

    protected override void OnStart()
    {
        Map.OnMapLoaded += SpawnPlayers;
    }

    private void SpawnPlayers()
    {
        if ( !Networking.IsHost ) return;

        foreach ( var connection in Connection.All )
            PlayerPrefab.Clone().NetworkSpawn( connection );
    }
}
```

### Server-authoritative map switch

```csharp
[Rpc.Host]
public void RequestMapChange( string mapName )
{
    if ( !Networking.IsHost ) return;
    Map.MapName = mapName;
    // Clients receive the change via snapshot replication of the property.
}
```

See [Map Networking](map-networking.md) for the full picture of how clients see this change.

- **Wrong package type.** If you give `MapName` a package ident whose `TypeName` is not `"map"`, the load is aborted and a warning is logged: `"Package … is not a map - it's a …"`.
- **Empty `MapName`.** Assigning `null` or whitespace unloads the current map and sets `loadedMapName` to null. `IsLoaded` becomes false.
- **Map name normalization.** Internally, `.vmap` is converted to `.vpk` immediately, so `loadedMapName` always reflects the `.vpk` form regardless of what you assigned. If you compare `MapName` against `loadedMapName` from external code, normalize both first.
- **Cancellation.** Long downloads can be cancelled mid-flight (e.g. by setting a new `MapName` or disabling the component). The token sources are tracked per-instance, so cancellation only affects loads belonging to that `MapInstance`.

## Related Pages

- [Maps overview](index.md)
- [Map Entities](map-entities.md)
- [Map Networking](map-networking.md)
- [Map Instance component reference](../components/reference/mapinstance.md)
