---
title: "Network Events"
icon: "📆"
created: 2023-11-25
updated: 2024-03-13
sources:
  - engine/Sandbox.Engine/Scene/Components/Markers/INetworkListener.cs
  - engine/Sandbox.Engine/Scene/Components/Markers/INetworkSpawn.cs
---

# Network Events

Your games will often need to react to network lifecycle events, like a player joining the game, disconnecting, or a networked object being spawned. You can hook into these events by implementing `Component.INetworkListener` or `Component.INetworkSpawn`.

## Quick Working Example

Here is a simple manager component that automatically spawns a player prefab when someone successfully loads into the game.

```csharp
public sealed class GameNetworkManager : Component, Component.INetworkListener
{
	[Property] public GameObject PlayerPrefab { get; set; }
	[Property] public GameObject SpawnPoint { get; set; }

	/// <summary>
	/// Called on the host when someone successfully joins the server (including the local player).
	/// </summary>
	public void OnActive( Connection connection )
	{
		// Spawn a player for this client
		var player = PlayerPrefab.Clone( SpawnPoint.Transform.World );

		// Spawn it on the network, assigning the connection as the owner
		player.NetworkSpawn( connection );
	}
}
```

### Authorizing Connections
You can use `AcceptConnection` to implement bans, password checks, or player limits. If any component returns `false`, the connection is rejected.

```csharp
public bool AcceptConnection( Connection channel, ref string reason )
{
    if ( channel.DisplayName == "TrollPlayer" )
    {
        reason = "You have been banned.";
        return false;
    }
    return true;
}
```

### Reacting to Network Spawns
When a networked object is spawned, you might want to initialize visuals or play a sound.

```csharp
public class SpawnEffect : Component, Component.INetworkSpawn
{
    public void OnNetworkSpawn( Connection owner )
    {
        Log.Info( $"This object was spawned! Owner: {owner.DisplayName}" );
    }
}
```

## Configuration & Callbacks

### INetworkListener Methods
These methods are typically only called on the **Host** (the player running the server).

| Method | Description |
|--------|-------------|
| `AcceptConnection` | Called to decide if a connection is allowed. Return `false` to reject. |
| `OnConnected` | The client has connected and is starting the handshake (downloading packages). |
| `OnActive` | The client is fully loaded, completed the handshake, and entered the game. |
| `OnDisconnected` | The client has disconnected from the server. |
| `OnBecameHost` | Called when the previous host leaves, and you are migrated to be the new host. |

### INetworkSpawn Methods

| Method | Description |
|--------|-------------|
| `OnNetworkSpawn` | Called when this object is spawned on the network. Passes the `Connection` of the owner. |

## Troubleshooting

:::warning Listeners Firing Only on Host
A common mistake is expecting `INetworkListener` methods like `OnActive` or `OnDisconnected` to run on all clients. **These events only fire on the Host.** If you need to tell all clients that a player joined, the Host must receive the `OnActive` event and then use an [RPC Message](rpc-messages.md) to notify the other clients.
:::

:::danger Missing Scene Components
For `INetworkListener` events to fire, the component must actually exist on an active GameObject in your current Scene. If it's on a prefab in your asset browser, it won't do anything.
:::

## Related Pages
* [Networked Objects](networked-objects.md)
* [RPC Messages](rpc-messages.md)
