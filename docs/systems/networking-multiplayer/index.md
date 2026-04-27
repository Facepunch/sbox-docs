---
title: "Networking & Multiplayer"
icon: "🧑‍🤝‍🧑"
created: 2023-11-24
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Systems/Networking/Networking.cs
---

# Networking & Multiplayer

s&box uses host-client networking. One player is the host (the authoritative server, usually their own game), the rest are clients that connect to them. Steam P2P handles the transport for casual multiplayer; dedicated servers handle the rest.

If you've shipped a multiplayer Unity or Unreal game, the model maps cleanly: `[Sync]` ≈ replicated variable, `[Rpc.Broadcast]` ≈ multicast RPC.

## How it Works

S&box networking is **authoritative host-client**. 

*   **One Codebase:** Unlike engines that split code into `.server.cs` and `.client.cs`, the same C# class runs on every machine. You use properties like `IsProxy` to decide if the current machine "owns" the object or is just a remote viewer.
*   **The Host:** The host (the player who started the game or a dedicated server) is the source of truth. They simulate physics and own the state.
*   **Snapshots:** The host takes a "snapshot" of the world state (every `[Sync]` property and every GameObject transform) multiple times per second and sends it to everyone else.
*   **Proxies:** On a client's machine, most objects are "Proxies." They don't run their own physics; they just smoothly interpolate between the snapshots received from the host.

## A working example

```csharp
using Sandbox;
using Sandbox.Network;

public sealed class LobbyManager : Component
{
    protected override void OnStart()
    {
        Networking.CreateLobby( new LobbyConfig
        {
            MaxPlayers = 8,
            Privacy = LobbyPrivacy.Public,
            Name = "My Awesome Lobby"
        } );
    }

    public void SpawnPlayer( GameObject playerPrefab, Transform spawnPoint, Connection connection )
    {
        var go = playerPrefab.Clone( spawnPoint );
        go.NetworkSpawn( connection );    // assigns the spawning client as owner
    }
}
```

Hosting a lobby and spawning a player is roughly that simple. The pieces below are what you call into for the harder cases.

## The four pieces

### `NetworkSpawn` — make a GameObject visible to everyone

A GameObject only exists locally until you network it.

```csharp
go.NetworkSpawn();              // unowned (host simulates)
go.NetworkSpawn( connection );  // owned by a specific client
```

Once networked, the GameObject's transform syncs automatically and it can use the rest of the networking features. Read more in [Networked Objects](networked-objects.md).

### `[Sync]` — replicate a property's value

```csharp
public class PlayerStats : Component
{
    [Sync] public int Health { get; set; } = 100;
}
```

Only the **owner** of the GameObject (or the host on unowned objects) is allowed to write a `[Sync]` property. Clients that aren't the owner write to it locally and the value gets overwritten by the next snapshot. Always guard writes with `if ( IsProxy ) return;`.

Use `[Sync]` for state — health, scores, inventory contents, the door is open. See [Sync Properties](sync-properties.md) for `SyncFlags`, `[Change]` callbacks, and `NetList<T>` / `NetDictionary<K,V>`.

### `[Rpc.*]` — call a function on other peers

```csharp
[Rpc.Broadcast]
public void PlayJumpEffect()
{
    Sound.Play( "jump.sound", WorldPosition );
}
```

RPCs are for transient events — sounds, effects, ephemeral notifications. Variants:
- `[Rpc.Broadcast]` — call on every peer
- `[Rpc.Host]` — only the host runs it
- `[Rpc.Owner]` — only the owner of the GameObject runs it

Read [RPC Messages](rpc-messages.md) for filtering, arguments, and the `Rpc.Caller` accessor.

### `Ownership` — who simulates what

The owner of a networked GameObject runs its physics and writes its `[Sync]` properties. By default the host is the owner; pass a `Connection` to `NetworkSpawn` to give ownership to a specific client. Ownership can change at runtime via `Network.SetOwnerTransfer()`. See [Ownership](ownership.md).

## When to use which

| You want to... | Use |
|---|---|
| Tell everyone "the player jumped" | `[Rpc.Broadcast]` |
| Keep player health in sync | `[Sync]` |
| Spawn a bullet from one client and see it on all peers | `Clone().NetworkSpawn()` |
| React when health changes | `[Sync, Change(nameof(OnHealthChanged))]` |
| Block clients from forging health values | `[Sync(SyncFlags.FromHost)]` (only host writes) |
| Have the host pick which player just won | `Networking.IsHost` check + `[Rpc.Broadcast]` |

State → `[Sync]`. Events → RPCs. The mistake people make early is using RPCs to keep state in sync ("RPC the new health to everyone every time it changes"); use `[Sync]` instead, it handles late-joiners and snapshot replay correctly.

## Things that bite

**Late-joining clients miss past RPCs.** They get the current snapshot (so all `[Sync]` properties arrive at correct values) but they don't get RPCs that fired before they connected. Door state should be `[Sync]`; the door-opening sound is the RPC.

**RPCs to a non-networked GameObject silently no-op.** Always `NetworkSpawn` before sending RPCs from a component on that object.

**Adding a component after `NetworkSpawn` doesn't network it.** Call `GameObject.Network.Refresh()` after structural changes.

**Host-leaves disconnects everyone.** s&box doesn't auto-migrate hosts in P2P. If your game can survive losing the host, plan around it; otherwise look at [Dedicated Servers](dedicated-servers/index.md).

## Where to go next

Reference pages:

- [Networked Objects](networked-objects.md)
- [Sync Properties](sync-properties.md)
- [RPC Messages](rpc-messages.md)
- [Ownership](ownership.md)
- [Network Events](network-events.md) — `OnActive`, `OnDisconnected`, etc.
- [Network Visibility](network-visibility.md) — bandwidth culling for large maps
- [Custom Snapshot Data](custom-snapshot-data.md) — when you need to control snapshot serialization
- [HTTP Requests](http-requests.md) and [WebSockets](websockets.md) — talking to non-game servers
- [Connection Permissions](connection-permissions.md) — fine-grained access control
- [Dedicated Servers](dedicated-servers/index.md) — non-P2P hosting
- [Testing Multiplayer](testing-multiplayer.md) — running multiple local clients

Practical:

- [Network Helper](network-helper.md) — drop-in lobby + player-spawn helper from the `base` addon
- [Sync Variables (recipe)](../../how-to/sync-variables.md) — minimal copy-paste recipe
- [Build a Networked Lobby (tutorial)](../../tutorials/networking-basics.md) — end-to-end walkthrough
