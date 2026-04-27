---
title: "Map Networking"
icon: "🌐"
created: 2026-04-25
updated: 2026-04-27
sources:
  - engine/Sandbox.Engine/Scene/Components/Map/MapInstance.cs
  - engine/Sandbox.Engine/Game/LaunchArguments.cs
  - engine/Sandbox.Engine/Game/Game/Game.cs
  - engine/Sandbox.Engine/Scene/Networking/NetworkObject.cs
---

# Map Networking

> **New to maps?** The Maps overview is at **[Maps](index.md)**. For authoring (the **Mapping** tool), see **[Scene Mapping](../scene-mapping/index.md)**. This page is the reference for how map loading is coordinated across the network.

In a multiplayer game, a map change must be coordinated across the server and every connected client. `MapInstance` participates in the snapshot/network system in three ways:

1. **`MapName` is a regular `[Property]`** — it serializes into the scene snapshot, so when a client receives the snapshot, it sees the host's current `MapName` and loads the same map.
2. **`MapInstance.UnloadMap` respects network ownership** — child GameObjects spawned from snapshot data are not deleted by the client during unload.
3. **`LaunchArguments.Map` is what the menu pushes when you click "Play"** — and it survives reconnect via `Game.cs` populating `ReconnectMsg.Map`.

## What's actually networked

Most of a map is **not** networked — it's loaded identically from the same `.vpk`/`.scene` file on every peer, so there's nothing to replicate. The only entity type that is automatically network-spawned is `prop_physics`, which becomes a fully networked `Prop` so any client can push, shoot, or destroy it and have the change replicate.

Loaded locally on each peer (deterministic — same file → same result):

- World geometry and collision
- Lights and light probes (cubemaps, env probes)
- Fog volumes (gradient, cubemap, volumetric)
- Skyboxes (2D + 3D)
- Soundscapes
- Static props (`prop_dynamic` / `prop_animated`) and `func_brush` entities

Sent via the snapshot system:

- The `MapInstance.MapName` value itself (so clients know which map to load)
- Each `prop_physics` GameObject after spawn
- Any GameObjects you yourself `NetworkSpawn` from a custom map entity handler

That's the full split. If you author a custom entity that needs to replicate state, you have to opt it in explicitly — the map loader does not network unrecognized entities for you.

## How a Client Sees a Map Load

When a client connects to a host:

1. The host serializes the scene to a snapshot. The `MapInstance` component's `MapName` is included as a property value.
2. The snapshot is sent to the client.
3. On the client, `MapInstance` is reconstructed with `MapName` set to the host's value.
4. `OnUpdate` notices `loadedMapName != MapName` and starts the same async load locally.
5. `OnMapLoaded` fires on the client when its local copy is ready. The host has typically already finished by this point.

This means **clients load the map themselves** — the actual map data is not streamed from the host. Each client must already have the map package mounted (either via Steam download triggered by `Package.Fetch`, or because it was bundled with the game / a mounted addon).

## Server-Driven Map Changes

To change the map mid-session, the host sets `MapName` on the `MapInstance`:

```csharp
[Rpc.Host]
public void RequestMapChange( string mapName )
{
    if ( !Networking.IsHost )
        return;

    var map = Components.Get<MapInstance>();
    map.MapName = mapName;
}
```

Because `MapName` is a `[Property]`, the change replicates via the next snapshot delta. Clients see the new value, their `MapInstance.OnUpdate` sees the mismatch with `loadedMapName`, and the local load kicks off. Each client fires its own `OnMapLoaded` independently when its local load completes.

> The `MapName` property has no special networking annotation — it relies on the standard scene-property snapshot mechanism. There is no built-in "wait for all clients to finish loading" gate; if you need that synchronization, build it explicitly with RPCs.

## Unload Behavior on Clients

`MapInstance.UnloadMap` walks the map's child GameObjects and deletes them — but with two safety checks:

```csharp
foreach ( var child in GameObject.Children )
{
    // In editor, don't delete saved child objects
    if ( Scene.IsEditor && !child.Flags.Contains( GameObjectFlags.NotSaved ) )
        continue;

    // If I'm a client and this came from the snapshot, don't delete it
    if ( Networking.IsClient && child.NetworkMode == NetworkMode.Snapshot )
        continue;

    // If it's a fully networked object and I'm not the owner, leave it
    if ( !child.Network.Active || child.Network.IsOwner )
        child.Destroy();
}
```

What this means in practice:

- **Map-spawned GameObjects** (entities placed in Hammer) are flagged `GameObjectFlags.NotSaved` and have their lifetime tied to the `MapInstance` on every peer. They are deleted on every peer when the map unloads.
- **Network-spawned GameObjects** (e.g. `PrefabToSpawn.Clone().NetworkSpawn()`) are *not* destroyed by a non-owning client during map unload. The host (the owner) is responsible; clients leave them alone and let the snapshot drive deletion.
- **Snapshot-only GameObjects** (`NetworkMode.Snapshot`) on a client are explicitly skipped — the snapshot system manages their lifetime, not `MapInstance`.

This means you can safely have networked gameplay objects (players, projectiles) that *outlive* a map change, as long as the host re-parents or destroys them appropriately.

## Reconnect and Launch Arguments

When a client reconnects to a server (after a disconnect or a host map change that requires a full reload), the engine builds a `ReconnectMsg`:

```csharp
// engine/Sandbox.Engine/Game/Game/Game.cs:194
var msg = new ReconnectMsg { Game = gameIdent, Map = LaunchArguments.Map };
```

So `LaunchArguments.Map` carries the map identity across the reconnect. If your boot scene has `MapInstance.UseMapFromLaunch = true`, the new session loads the right map automatically.

This is also the path the main menu uses: clicking "Play" on a map populates `LaunchArguments.Map` before the game starts, and any `MapInstance` with `UseMapFromLaunch = true` picks it up on its first `OnLoad` / `OnUpdate`.

## Common Patterns

### Wait until every client has loaded

There is no built-in barrier; build one with an RPC:

```csharp
[Sync] public NetList<Guid> ReadyClients { get; set; } = new();

protected override void OnStart()
{
    Map.OnMapLoaded += () => ReportReady();
}

[Rpc.Host]
private void ReportReady()
{
    var caller = Rpc.Caller;
    if ( !ReadyClients.Contains( caller.Id ) )
        ReadyClients.Add( caller.Id );

    if ( ReadyClients.Count == Connection.All.Count )
        StartRound();
}
```

### Map switch from a UI button (host only)

```csharp
public void OnMapButtonClicked( string mapName )
{
    if ( Networking.IsHost )
    {
        // Host-side: just set it
        Map.MapName = mapName;
    }
    else
    {
        // Client-side: send an RPC and let the host decide
        RequestMapChange( mapName );
    }
}
```

- **Property hiding during load.** While a load is in flight on a client, `MapName` already equals the host's value but `IsLoaded` is still false. UI that branches on map state should look at `IsLoaded`, not `MapName`.
- **Different map versions.** If two clients have different versions of the same map package mounted, the snapshot will tell them to load the same `MapName` but the resulting `SceneMap`s may differ. There is no built-in version mismatch detection.
- **Mid-load disconnect.** If a client disconnects mid-load, its in-flight `LoadMapAsync` is cancelled when the `MapInstance` is destroyed (its tracked cancellation tokens are cancelled by `CancelLoading`).
- **Editor-mode networking.** In the editor, `Scene.IsEditor` is true and `MapInstance.UnloadMap` skips deletion of saved children. Networking is not active in the editor unless you explicitly start a session, so the snapshot/owner branches don't apply.

## Related Pages

- [Maps overview](index.md)
- [Loading Maps](loading-maps.md)
- [Map Entities](map-entities.md)
- [Networked Objects](../../systems/networking-multiplayer/networked-objects.md)
- [Sync Properties](../../systems/networking-multiplayer/sync-properties.md)
- [Custom Snapshot Data](../../systems/networking-multiplayer/custom-snapshot-data.md)
