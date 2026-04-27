---
title: "RPC Messages"
icon: "✉️"
created: 2023-11-25
updated: 2024-11-29
sources:
  - engine/Sandbox.Engine/Systems/Networking/Networking.cs
---

# RPC Messages

> **New to networking?** Read **[Networking & Multiplayer](index.md)** first — that's the overview. This is the reference for one piece (RPCs); the systems landing covers the full picture.

Remote Procedure Calls (RPCs) allow you to call a method on one machine and have it execute across the network on other machines.

## Quick Working Example

```csharp
public sealed class Door : Component
{
	public void OpenDoor()
	{
		// ... run local door opening logic ...
		
		// Tell everyone else to play the sound
		PlayDoorSound( "sounds/door_open.sound", WorldPosition );
	}

	[Rpc.Broadcast]
	public void PlayDoorSound( string soundName, Vector3 position )
	{
		Sound.Play( soundName, position );
	}
}
```

### Broadcasting to Everyone

The most common RPC is `[Rpc.Broadcast]`, which executes the method on the Host and all connected Clients.

```csharp
[Rpc.Broadcast]
public void ShowHitMarker()
{
	// This will run on every machine
	Log.Info( "Someone got hit!" );
}
```

### Static RPCs

RPCs don't have to be tied to a specific Component instance. You can create static RPCs that can be called from anywhere.

```csharp
[Rpc.Broadcast]
public static void SendChatMessage( string message )
{
	ChatBox.AddMessage( message );
}
```

### Identifying the Caller

You often need to know *who* sent the RPC (e.g., to verify they are allowed to perform the action). You can use `Rpc.Caller` to get the `Connection` of the sender.

```csharp
[Rpc.Broadcast]
public void BroadcastMessage( string text )
{
	Log.Info( $"{Rpc.Caller.DisplayName} says: {text}" );
}
```

## Configuration & Routing

You control *who* receives the RPC by using different attributes:

| Attribute | Description |
|-----------|-------------|
| `[Rpc.Broadcast]` | Executes on the Host and every connected Client. |
| `[Rpc.Owner]` | Executes **only** on the specific Client that owns the GameObject (or the Host if unowned). |
| `[Rpc.Host]` | Executes **only** on the Host. Useful for clients sending requests to the server. |

### Network Flags

You can further configure how the message is transmitted using `NetFlags`:

```csharp
// Send unreliably for maximum speed (good for visual effects)
[Rpc.Broadcast( NetFlags.Unreliable )]
public void SpawnSparks() { ... }
```

| Flag | Description |
|------|-------------|
| `NetFlags.Reliable` | *(Default)* Guaranteed delivery, in order. Slower, but safe. Use for gameplay events. |
| `NetFlags.Unreliable` | Faster delivery, but packets might be dropped or arrive out of order. Use for visual/audio effects. |
| `NetFlags.SendImmediate` | Sent instantly instead of batching with other messages. Use for critical timing (like voice chat). |
| `NetFlags.DiscardOnDelay` | Drops the message if it can't be sent quickly. Only applies to unreliable messages. |
| `NetFlags.UnreliableNoDelay` | Composite of `Unreliable \| SendImmediate \| DiscardOnDelay` — minimum-latency, drop-if-late. |
| `NetFlags.HostOnly` | Restricts the RPC so it can *only* be called by the Host. |
| `NetFlags.OwnerOnly` | Restricts the RPC so it can *only* be called by the owner of the GameObject. |

## Advanced Usage

### Filtering Recipients

Sometimes you want to broadcast an RPC to everyone *except* a specific player, or *only* to a specific team. You can wrap your RPC call in a filter block.

```csharp
// Send to everyone EXCEPT the player named Harry
using ( Rpc.FilterExclude( c => c.DisplayName == "Harry" ) )
{
	PlayDoorSound( "bing", WorldPosition );
}

// Send ONLY to the player named Garry
using ( Rpc.FilterInclude( c => c.DisplayName == "Garry" ) )
{
	PlayDoorSound( "bing", WorldPosition );
}
```

## Troubleshooting

:::warning Unsupported Argument Types
RPC arguments must be serializable by the network system. You can pass basic types (`int`, `float`, `string`, `Vector3`, `Rotation`) and engine types (`GameObject`, `Component`), but you cannot pass complex custom classes or dictionaries directly as arguments.
:::

## Related Pages
* [Networked Objects](networked-objects.md)
* [Sync Properties](sync-properties.md)
