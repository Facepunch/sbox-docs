---
title: Networking & Multiplayer
icon: "☁️"
created: 2026-04-25
updated: 2026-04-25
---

# Networking & Multiplayer

Things that "work in the editor / break in multiplayer." Sync, RPC, ownership, network spawn, dedicated servers.

## Sync and ownership

### "My synced variable isn't updating for other players"

The most common cause: you're writing to it from a non-owner. `[Sync]` only replicates writes from the object's owner.

- **Check:** wrap the write in `if ( IsProxy ) return;` — if writes stop happening, you weren't the owner.
- **Check:** is the GameObject actually networked? Call `NetworkSpawn()` once or set Network Mode to `Object` in the Inspector.
- **Less common:** you're mutating a private backing field directly (the setter never runs). See [Sync Properties](../systems/networking-multiplayer/sync-properties.md).

**Deep dive:** [Sync Properties](../systems/networking-multiplayer/sync-properties.md), [Ownership](../systems/networking-multiplayer/ownership.md).

### "My object's position isn't updating for other players"

The object is local-only. Networking is opt-in per object.

- **Fix:** set Network Mode to `Object` in the Inspector, **or** call `go.NetworkSpawn()` after instantiating.
- **Fix:** check `IsOwner` before writing. Non-owners writing to networked transforms get rubber-banded by the host.

**Deep dive:** [Networked Objects](../systems/networking-multiplayer/networked-objects.md).

### "My object is stuttering or rubber-banding"

Two writers fighting for authority. The host keeps correcting clients who don't own the object.

- **Fix:** only the owner writes the transform. Everyone else reads.
- **Fix:** call `obj.Network.TakeOwnership()` from the player who should drive it (and only one).

**Deep dive:** [Ownership](../systems/networking-multiplayer/ownership.md).

### "I added a component to a networked object, others don't see it"

Hierarchy/component changes are not auto-networked after `NetworkSpawn`.

- **Fix:** `GameObject.Network.Refresh()` after adding/removing components.

## Network spawn and lifecycle

### "NetworkSpawn failed: Object not registered"

You called `NetworkSpawn()` before the object was fully constructed and active.

- **Fix:** call it after components are added and the object is active in the Scene.
- **Order:** Clone → set up → activate → `NetworkSpawn()`.

**Deep dive:** [Server Architecture](../systems/networking-multiplayer/server-architecture.md).

### "My player spawned locally, but other clients can't see them"

Player Prefab isn't a Network Object, or you didn't pass the connection.

- **Fix:** set `NetworkMode = NetworkMode.Object` on the prefab.
- **Fix:** `playerObj.NetworkSpawn( connection )` so the Connection becomes the owner.

**Deep dive:** [Build a Networked Lobby](../tutorials/networking-basics.md).

## RPCs

### "My RPC isn't doing anything"

The GameObject the RPC is on must be network-spawned.

- **Check:** the object has `NetworkMode.Object` and was spawned via `NetworkSpawn`.
- **Check:** RPC method signature uses supported types (basic types, `Vector3`, `Rotation`, `GameObject`, `Component`). Custom classes fail silently.

**Deep dive:** [RPC Messages](../systems/networking-multiplayer/rpc-messages.md).

### "My RPC argument types throw at runtime"

Only network-serializable types are allowed: primitives, engine value types, `GameObject`, `Component`. Lists of those work; arbitrary classes don't.

- **Fix:** flatten complex data into a struct of primitives, or use `[Sync]` properties instead of RPC arguments.

## Snapshots and custom data

### "INetworkSnapshot doesn't sync after the client joins"

`INetworkSnapshot` runs **only** when a client first joins. Later changes don't replicate automatically.

- **Fix:** for ongoing changes, use `[Sync]` properties or RPCs to push updates after join.

**Deep dive:** [Custom Snapshot Data](../systems/networking-multiplayer/custom-snapshot-data.md).

### "My snapshot read/write are crashing or producing garbage"

`WriteSnapshot` and `ReadSnapshot` must read fields in the **exact same order** and types. Off-by-one and the stream desyncs.

- **Fix:** read in the same order you wrote, with the same types. Write a unit test that round-trips a snapshot.

## Network events and listeners

### "My INetworkListener method only fires on the host"

By design. `INetworkListener` events (`OnActive`, `OnDisconnected`) fire on the **host only**.

- **Fix:** if you need to notify all clients, send an RPC from the host's listener.

**Deep dive:** [Network Events](../systems/networking-multiplayer/network-events.md).

### "Listener method isn't firing at all"

The listener must be on a Component on an *active GameObject in the current Scene*. Listeners on prefabs in the asset browser don't run.

- **Fix:** put the listener on a manager GameObject that exists in your startup scene.

## Visibility and proxies

### "My object disappears immediately when AlwaysTransmit is off"

Without `AlwaysTransmit` and without an `INetworkVisible` implementation that returns `true`, the object is culled for every client.

- **Fix:** keep `AlwaysTransmit` on, **or** implement `INetworkVisible.IsVisibleToConnection` and return `true` selectively.
- **Note:** `IsVisibleToConnection` only runs on the host.

**Deep dive:** [Network Visibility](../systems/networking-multiplayer/network-visibility.md).

## Lobby and connection

### "Players can't join my lobby"

`LobbyConfig.Privacy = LobbyPrivacy.FriendsOnly` blocks public discovery.

- **Fix:** set Privacy to `Public` for open lobbies.
- **Check:** firewall isn't blocking the connection ports.

**Deep dive:** [Networking Index](../systems/networking-multiplayer/index.md).

### "Connection refused on `connect local`"

Your primary instance isn't acting as a host yet.

- **Fix:** press Play in the primary editor to start the host.

**Deep dive:** [Testing Multiplayer](../systems/networking-multiplayer/testing-multiplayer.md).

### "Server is running a different version"

Server's compiled addon hash doesn't match the client's. Auto-disconnect kicks in.

- **Fix:** make sure the dedicated server pulled the latest addon code/assets and recompiled before clients try to connect.

**Deep dive:** [Client Architecture](../systems/networking-multiplayer/client-architecture.md).

### "Players can't connect to my dedicated server over the internet"

Port forwarding. s&box dedicated servers default to UDP/TCP **27015**.

- **Fix:** open 27015 on your router (and any firewall on the host machine).

**Deep dive:** [Dedicated Servers](../systems/networking-multiplayer/dedicated-servers/index.md).

### "Works in editor, breaks on dedicated server"

In editor you're host *and* client. On dedicated, the server has no client. Code that assumes "I am a player" runs on the server and breaks.

- **Fix:** guard authority-sensitive code with `if ( Networking.IsHost )` and client-only code with `if ( !Application.IsHeadless )`.

## HTTP and WebSockets

### "My Http request never finishes"

You forgot `await`. Async methods return immediately if not awaited.

- **Fix:** `var result = await Http.RequestStringAsync( url );`
- **Check:** the URL is allowed by the engine's HTTP rules (no raw IPs, no non-standard localhost ports unless `-allowlocalhttp` is passed).

**Deep dive:** [HTTP Requests](../systems/networking-multiplayer/http-requests.md).

### "Access to 'http://127.0.0.1/api' is not allowed"

The engine blocks raw IP addresses by default for security.

- **Fix:** use `localhost` on a standard port like 8080.
- **Fix:** for local testing, launch with `-allowlocalhttp`.

### "WebSocket: Cannot access a disposed object"

You called `Send` after `Dispose`, or after the Component was destroyed.

- **Fix:** null-check the socket and check `IsValid` before sending.
- **Fix:** dispose only in `OnDestroy`, not in random callbacks.

**Deep dive:** [WebSockets](../systems/networking-multiplayer/websockets.md).

### "WebSocket connection refused"

URI must be `ws://` or `wss://`. Plain HTTP servers won't upgrade to WebSocket.

- **Fix:** verify the server actually speaks WebSocket on the path you're hitting.

## Snapshot too large

### "Snapshot too large — connection drops"

A single server snapshot blew the UDP packet ceiling. Usually caused by suddenly creating thousands of networked objects.

- **Fix:** stagger object spawns. Use `INetworkVisible` to send only what's relevant per client.
- **Engine-level:** delta compression should handle most cases — if you're seeing this routinely, look at what's getting added in bulk.

## Related Pages

- [Scene & Components](./scene-and-components.md)
- [Physics](./physics.md)
- [Runtime & Build](./runtime-and-build.md)
