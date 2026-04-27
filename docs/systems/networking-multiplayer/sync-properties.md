---
title: "Sync Properties"
icon: "🔄"
created: 2024-01-10
updated: 2026-04-10
sources:
  - engine/Sandbox.Engine/Scene/Networking/Sync.cs
  - engine/Sandbox.Engine/Scene/Networking/SyncFlags.cs
---

# Sync Properties

> **New to networking?** Read **[Networking & Multiplayer](index.md)** first — that's the overview. This is the reference for one piece (`[Sync]` properties); the systems landing covers the full picture.

The `[Sync]` attribute automatically shares a property's value across the network, keeping all players in sync when the owner changes it.

## Quick Example

```csharp
public class PlayerStats : Component
{
	// This will automatically update for everyone when the owner changes it
	[Sync] public int Kills { get; set; }

	public void AddKill()
	{
		// Only the owner (or host) can change synced properties
		if ( IsProxy ) return;
		
		Kills++; 
	}
}
```

## How It Works

Think of `[Sync]` like a one-way radio broadcast. The owner of the networked object holds the microphone. When they change the value of a `[Sync]` property, the network system automatically broadcasts that new value to every other client (the listeners). 

Because it's a one-way street, only the **owner** of the object (or the host, if nobody owns it) is allowed to change the value. If another client tries to change it, their change will be ignored or overwritten by the owner's true value.

Any unmanaged value type (like `int`, `bool`, `float`, `Vector3`, `Rotation`) can be synced. It also supports `string`, as well as specific engine types like `GameObject`, `Component`, and `GameResource`.

## Usage Patterns

### Detecting when a value changes

Often, you want to trigger some visual effect or logic when a synced value updates (like playing a sound when health drops). You can use the `[Change]` attribute alongside `[Sync]` to call a method whenever the value changes.

```csharp
public class Flashlight : Component 
{
	// When IsOn changes, the OnPowerToggled method will be called automatically
	[Sync, Change( nameof(OnPowerToggled) )] public bool IsOn { get; set; }
	
	private void OnPowerToggled( bool oldValue, bool newValue )
	{
		// Turn on/off a light component based on the new value
		Log.Info( $"Flashlight was turned {(newValue ? "on" : "off")}!" );
	}
}
```

### Syncing Collections

If you need to sync a list of items or a dictionary, standard C# `List<T>` won't work because the network system can't track when items are added or removed inside it. Instead, use `NetList<T>` and `NetDictionary<K,V>`.

```csharp
public class Inventory : Component 
{
	// Initialize it right away. The network system handles syncing the contents.
	[Sync] public NetList<string> Items { get; set; } = new();

	public void GiveStarterItems()
	{
		if ( IsProxy ) return;
		
		Items.Add( "Pistol" );
		Items.Add( "HealthKit" );
	}
}
```

*(Note: `[Change]` callbacks do not currently fire when an item is added/removed from a `NetList` or `NetDictionary`.)*

### Query Mode for Complex Getters/Setters

By default, the network system uses invisible code generation to detect when you call a property's `set` accessor. However, if your property just wraps a private backing field and you sometimes modify that field *directly*, the network system won't know it changed. Use `SyncFlags.Query` to force the system to manually check the value every tick.

```csharp
public class MovingObject : Component
{
	private Vector3 _velocity;

	// Query mode checks the value of _velocity every network tick to see if it changed
	[Sync( SyncFlags.Query )]
	public Vector3 Velocity
	{
		get => _velocity;
		set => _velocity = value;
	}

	public void UpdateVelocity( Vector3 newVelocity )
	{
		// Modifying the backing field directly! 
		// Because we used SyncFlags.Query, this change WILL still be networked.
		_velocity = newVelocity;
	}
}
```

## Configuration Flags

You can customize exactly how a property behaves by passing `SyncFlags` into the `[Sync]` attribute:

| Flag | Description |
|------|-------------|
| `SyncFlags.Query` | Continually checks the property for changes instead of relying on the setter. Slower, but necessary if the value changes behind the scenes. |
| `SyncFlags.FromHost` | Ownership is overridden: **only the host** is allowed to change this value, even if another player owns the GameObject. |
| `SyncFlags.Interpolate` | The value smoothly transitions between updates for other clients, rather than snapping instantly. Supported types: `float`, `double`, `Angles`, `Rotation`, `Transform`, `Vector3`. |

## Troubleshooting

**"My synced property isn't updating for other players!"**
* Are you changing the property on the **owner** of the object? If you are a client and you change a property on an object you don't own (like a map prop), the network system will ignore it. Use `if ( IsProxy ) return;` to prevent non-owners from trying to run code that changes synced variables.
* Did you modify a private backing field directly? Make sure you assign the property itself (`MyProperty = 5`), or use `[Sync( SyncFlags.Query )]`.

**"My `[Change]` method isn't being called!"**
* Make sure the method signature is correct. It must take exactly two arguments matching the type of your property: `void MethodName( T oldValue, T newValue )`.
* Are you modifying a `NetList`? `[Change]` currently only fires when the entire list is reassigned to a brand new `NetList`, not when individual elements are added or removed.

## Related Pages

* [Networked Objects](networked-objects.md)
* [RPC Messages](rpc-messages.md)
* [Ownership](ownership.md)
