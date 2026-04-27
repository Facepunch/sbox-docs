---
title: "Network Helper"
icon: "🔌"
created: 2026-04-25
updated: 2026-04-25
sources:
  - game/addons/base/code/Components/Networking/NetworkHelper.cs
  - engine/Sandbox.Engine/Scene/Components/Markers/INetworkListener.cs
---

# Network Helper

> **New to networking?** Read **[Networking & Multiplayer](index.md)** first — that's the overview. This is the reference for the `NetworkHelper` drop-in component; the systems landing explains where it fits.

`NetworkHelper` is a small turnkey component that ships in the **base** addon. Drop it on a GameObject in your boot scene and it will: open a multiplayer lobby on the host, listen for clients joining, and spawn a player prefab for each one at a chosen spawn point. It is a convenience helper — *not* a hard requirement of the engine — and is intentionally simple enough to copy-paste and replace with your own equivalent.

## Quick Working Example

1. Create a GameObject called `Network` in your scene.
2. Add the `Network Helper` component (it lives under the **Networking** category).
3. Assign your player prefab to **Player Prefab**.
4. Optionally assign a few GameObjects to **Spawn Points**, or scatter `SpawnPoint` components in the scene.
5. Press play.

```csharp
// Programmatic equivalent — usually you'd do this in the editor.
var go = new GameObject( true, "Network" );
var net = go.Components.Create<NetworkHelper>();
net.PlayerPrefab = playerPrefab;       // GameObject
net.StartServer  = true;               // create a lobby if one isn't already active
```

When the scene loads:

- The host's `NetworkHelper.OnLoad` calls `Networking.CreateLobby` (unless networking is already active or `StartServer` is false). The loading screen reads `"Creating Lobby"` while it does.
- For every client that finishes connecting, `OnActive(Connection)` fires *on the host*, which clones `PlayerPrefab` at a spawn location and calls `NetworkSpawn(channel)` so the new client owns its player object.

## Properties

| Property | Type | Default | Purpose |
|---|---|---|---|
| `StartServer` | `bool` | `true` | If true, the helper opens a lobby in `OnLoad` when no networking session is active. Set to `false` in scenes that are joined into an existing session. |
| `PlayerPrefab` | `GameObject` | `null` | The prefab cloned for each connected player. If unset, `OnActive` returns early and no player is spawned. |
| `SpawnPoints` | `List<GameObject>` | `null` | Optional list of spawn-point GameObjects. When populated, takes precedence over scene-wide `SpawnPoint` components. |

## When to Use It vs. Roll Your Own

`NetworkHelper` is intentionally minimal:

- It only handles `OnActive` — it does not respond to `OnDisconnected`, `AcceptConnection`, or `OnBecameHost`.
- It assumes one player object per connection, spawned immediately.
- It uses a uniform random pick across spawn points (no team logic, no anti-camping).

Use it when you're prototyping or when the defaults match your needs. Replace it with your own `Component.INetworkListener` when you need any of:

- A custom spawn algorithm (team-based, role-based, deferred until ready).
- A connection accept/reject policy (implement `AcceptConnection`).
- Cleanup on disconnect (implement `OnDisconnected`).
- A different lobby creation flow (custom `LobbyConfig`, not the default `new()`).

A drop-in replacement looks like this:

```csharp
public sealed class MyNetworkBoss : Component, Component.INetworkListener
{
    [Property] public GameObject PlayerPrefab { get; set; }

    protected override async Task OnLoad()
    {
        if ( Scene.IsEditor || Networking.IsActive ) return;
        Networking.CreateLobby( new() );
    }

    public bool AcceptConnection( Connection channel, ref string reason )
    {
        if ( IsBanned( channel.SteamId ) ) { reason = "banned"; return false; }
        return true;
    }

    public void OnActive( Connection channel )
    {
        var player = PlayerPrefab.Clone( name: $"Player - {channel.DisplayName}" );
        player.NetworkSpawn( channel );
    }

    public void OnDisconnected( Connection channel )
    {
        // tear down per-player state
    }
}
```

- **Editor preview.** `OnLoad` early-outs when `Scene.IsEditor` is true, so opening a scene in the editor does **not** create a lobby — the helper only acts when the scene runs in-game.
- **Already in a session.** `StartServer` is bypassed if `Networking.IsActive` is true. A scene with `NetworkHelper.StartServer = true` is therefore safe to load while joining an existing lobby — it won't try to host.
- **Missing prefab.** If `PlayerPrefab.IsValid()` is false, `OnActive` logs the join but spawns nothing. Useful for spectator-only or menu scenes that still want to react to joins.
- **Host migration.** `NetworkHelper` does not implement `OnBecameHost`. After a host migration, the new host will not retroactively spawn players — they're already spawned. If you need migration-aware behaviour, implement it yourself.
- **Spawning twice.** `OnActive` fires once per connection on the host. Don't also subscribe to `Networking.OnConnected` and spawn there — you'll get duplicates.

## Related Pages

- [Networked Objects](networked-objects.md)
- [Ownership](ownership.md)
- [Network Events](network-events.md) — `Component.INetworkListener` and friends
- [Connection Permissions](connection-permissions.md) — for `AcceptConnection` policies
- [Spawn Point](../../scene/components/reference/spawnpoint.md)
