---
title: "Networked Objects"
icon: "👾"
created: 2024-03-21
updated: 2024-04-30
sources:
  - engine/Sandbox.Engine/Systems/Networking/Networking.cs
---

# Networked Objects

> **New to networking?** Read **[Networking & Multiplayer](index.md)** first — that's the overview. This is the reference for one piece (`NetworkSpawn` / `NetworkMode`); the systems landing covers the full picture.

Networked GameObjects automatically sync their existence, hierarchy, and properties across the server and all connected clients.

## Quick Working Example

```csharp
public sealed class SpawnerComponent : Component
{
    [Property] public GameObject PrefabToSpawn { get; set; }

    public void SpawnNetworkedItem()
    {
        // 1. Clone the prefab locally
        var item = PrefabToSpawn.Clone( GameObject.WorldTransform );

        // 2. Tell the network to spawn it for everyone
        item.NetworkSpawn();
    }
}
```

### Spawning with an Owner

When spawning an object, you can assign it an owner (a specific `Connection`). This is crucial for player characters, as the owner is granted authority to simulate its physics and change its synced properties.

```csharp
public void SpawnPlayer( Connection clientConnection )
{
    var player = PlayerPrefab.Clone();
    
    // Spawn the object and assign the specific client as the owner
    player.NetworkSpawn( clientConnection );
}
```

### Changing Network Modes

You can control exactly how a networked object is sent to clients using its `NetworkMode`. This can be configured in the Editor Inspector on the GameObject, or via code.

```csharp
// Change this object so it is only networked as part of the initial scene snapshot
GameObject.NetworkMode = NetworkMode.Snapshot;
```

## Configuration

| Mode | Behaviour |
|------|-----------|
| `NetworkMode.Object` | The GameObject is fully networked, supporting synchronized properties and RPCs. |
| `NetworkMode.Snapshot` *(default)* | The host sends this GameObject as part of the initial scene snapshot when a client joins. It does not actively sync changes afterwards. |
| `NetworkMode.Never` | This GameObject is never networked to other clients. |

## Advanced Usage

### Disabling Interpolation

By default, the transforms of all networked objects are smoothly interpolated for clients to hide network latency. Sometimes, like when teleporting an object, you need it to snap instantly.

```csharp
// Teleport the object
GameObject.WorldPosition = new Vector3( 1000, 0, 0 );

// Tell other clients to clear interpolation so it snaps immediately
GameObject.Network.ClearInterpolation();
```

You can also completely disable interpolation for an object:

```csharp
GameObject.Network.Interpolation = false;
```

### Refreshing Networked Objects

Once `NetworkSpawn()` is called, changes to the object's hierarchy (adding/removing components or children) are **not** automatically networked. If you significantly alter the hierarchy of a networked object, you must manually refresh it.

```csharp
var newVisuals = new GameObject( true, "Visuals" );
newVisuals.SetParent( GameObject );

// Tell the network to re-scan and sync the hierarchy changes
GameObject.Network.Refresh();
```

*(Note: By default, only the host can send refresh updates. You can change this via [Connection Permissions](connection-permissions.md).)*

## Troubleshooting

:::warning Changes Not Syncing
If you add a new Component to a networked object *after* it has been spawned, other clients will not see the new Component unless you call `GameObject.Network.Refresh()`.
:::

## Related Pages

* [Ownership](ownership.md)
* [Sync Properties](sync-properties.md)
* [RPC Messages](rpc-messages.md)
