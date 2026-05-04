---
title: "RPC Messages"
icon: "✉️"
created: 2023-11-25
updated: 2024-11-29
---

# RPC Messages

[Components](/scene/components/index.md) can contain RPCs. An RPC is a function that when called, is called remotely too.


Supported RPC arguments are the exact same as [Sync properties.](/networking/sync-properties.md) 

# Example

Imagine your game has a button, and you want it to make a bing noise when it's pressed. You could have a function like this.

```csharp
void OnPressed()
{
	Sound.Play( "bing", WorldPosition );
}
```

The problem here is, that sound is only played on the host, or on the client where OnPressed is called. You want everyone to hear that sound. So you instead do something like this.

```csharp
void OnPressed()
{
	PlayOpenEffects();
}

[Rpc.Broadcast]
public void PlayOpenEffects()
{
	Sound.Play( "bing", WorldPosition );
}
```

The attribute `[Rpc.Broadcast]` makes it so when you call that function, it broadcasts a network message to everyone to call that function too. Without any flags, anyone could technically call that function, but you could add flags to restrict it to only the host, or only the owner of the object. See the *Flags* section below for more details.


# Static RPC

Static methods can be RPCs, too. A static RPC does not need to exist on a `Component` but can exist as a method on any static class.

```csharp
[Rpc.Broadcast]
public static void PlaySoundAllClients( string soundName, Vector3 position )
{
	Sound.Play( soundName, position );
}
```

# `Rpc.Owner`

Unlike `[Rpc.Broadcast]` which calls the function for everybody, you can use `[Rpc.Owner]` instead which means that the function will only be called for the `Owner` of the networked object or the host if the object has no owner.

# `Rpc.Host`

Similarly to Rpc.Owner, adding this will mean the function is only called on the Host.

# Flags

When defining an RPC, you can define a number of flags.

```csharp
[Rpc.Broadcast( NetFlags.Unreliable | NetFlag.OwnerOnly )]
public static void PlaySoundAllClients( string soundName, Vector3 position )
{
  // ...
}
```

| Name | Description |
|------|-------------|
| `NetFlags.Unreliable` | Message will be sent unreliably. It may not arrive and it may be received out of order. But chances are that it will arrive on time and everything will be fine. This is good for sending position updates,  or spawning effects. This is the fastest way to send a message. It is also the cheapest. |
| `NetFlags.Reliable` | This is the default, so you don't need to specify this. Message will be sent reliably. Multiple attempts will be made until the recipient has received it. Use this for things like chat messages, or important events. This is the slowest way to send a message. It is also the most expensive. |
| `NetFlags.SendImmediate` | Message will not be grouped up with other messages, and will be sent immediately. This is most useful for things like streaming voice data, where packets need to stream in real-time, rather than arriving with a bunch of other packets. |
| `NetFlags.DiscardOnDelay` | Message will be dropped if it can't be sent quickly. Only applicable to unreliable messages. |
| `NetFlags.HostOnly` | This RPC can only be called from the Host. |
| `NetFlags.OwnerOnly` | This RPC can only be called from the owner of the object it's being called on. |

# Arguments

You can pass arguments to the RPC like any other method, and they'll get passed magically.

```csharp
void OnPressed()
{
	PlayOpenEffects( "bing", WorldPosition );
}

[Rpc.Broadcast]
public void PlayOpenEffects( string soundName, Vector3 position )
{
	Sound.Play( soundName, position );
}
```


# Filtering

You can filter the recipients of a *Broadcast* RPC. This allows you to exclude specific connections from receiving the RPC, or *only* include specific connections.


```csharp
// Don't send the RPC to player's called Harry (sorry Harry!)
using ( Rpc.FilterExclude( c => c.DisplayName == "Harry" ) )
{
	PlayOpenEffects( "bing", WorldPosition );
}

// Only send the RPC to player's called Garry.
using ( Rpc.FilterInclude( c => c.DisplayName == "Garry" ) )
{
	PlayOpenEffects( "bing", WorldPosition );
}
```

# Caller Information

You can check which connection called the method using the `Rpc.Caller` class.

```csharp
void OnPressed()
{
	PlayOpenEffects( "bing", WorldPosition );
}

[Rpc.Broadcast]
public void PlayOpenEffects( string soundName, Vector3 position )
{
	if ( !Rpc.Caller.IsHost ) return;

	Log.Info( $"{Rpc.Caller.DisplayName} with the steamid {Rpc.Caller.SteamId} played open effects!" );
	Sound.Play( soundName, position );
}
```
