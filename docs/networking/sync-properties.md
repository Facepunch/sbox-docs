---
title: "Sync Properties"
icon: "🔄"
created: 2024-01-10
updated: 2025-01-30
---

# Sync Properties

## Sync Attribute

Adding the `[Sync]` attribute to a property on a [Component](/scene/components/index.md) will have its latest value sent to other players each time it changes.


```csharp
public class MyComponent : Component
{
  [Sync] public int Kills { get; set; }
}
```

:::info
`[Sync]` only works when the GameObject has the `NetworkMode.Object` mode. Properties on `NetworkMode.Snapshot` objects are never synced after the initial snapshot to anyone, even if marked with `[Sync]`.
:::

These properties are controlled by the owner of the object, therefore only the owner of the object can change them. If you try to set a `[Sync]` property while `IsProxy` is `true`, the change is silently discarded - it will not replicate. For host-controlled values, see the `FromHost` flag below.


# Supported Types

`[Sync]` properties support unmanaged types, and string. You can't synchronize every class with them, but any value type including structs will be fine. `int`, `bool`, `Vector3`, `float` are all examples of valid types.  We also support serializing specific classes such as `GameObject`, `Component`, `GameResource`.


# Detecting Changes

You can detect changes to a `[Sync]` property by also applying a the `[Change]` attribute to it. With this attribute you can specify the name of a callback method that will be invoked when the value of the property has changed.



:::warning
The `[Change]` attribute will not invoke the callback when a collection has changed. The callback will only be invoked when the property itself is assigned to something different. To detect changes to collections, use the `OnChanged` event on `NetList<T>` and `NetDictionary<K,V>`. See the *Collections* section below for more details.
:::


```csharp
public class MyComponent : Component 
{
  [Sync, Change( "OnIsRunningChanged" )] public bool IsRunning { get; set; }
  
  private void OnIsRunningChanged( bool oldValue, bool newValue )
  {
    // The value of IsRunning has changed...
  }
}
```

You can use `nameof` to avoid hardcoding the method name as a string, making it safer to refactor:

```csharp
[Sync, Change( nameof(OnIsRunningChanged) )] public bool IsRunning { get; set; }
```


# Sync Flags

You can customize the behaviour of a synchronized property with `SyncFlags`.



| Flag | **Description** |
|------|-------------|
| `SyncFlags.Query` | Enables Query Mode for the property. See the *Query Mode* section below. |
| `SyncFlags.FromHost` | The host has ownership over the value, instead of the owner of the networked object. Only the host may change the value. |
| `SyncFlags.Interpolate` | The value of the property will be interpolated for other clients. The value is interpolated over a few ticks. |

### SyncFlags.FromHost

Use `FromHost` for values that represent shared game state - things every player sees the same way, and only the host should be authoritative about. A round timer or team score is a good example.

```csharp
// Only the host can set this; clients read it as-is
[Sync( SyncFlags.FromHost )] public int RoundTimer { get; set; }
```

### SyncFlags.Interpolate

Use `Interpolate` for properties that change every frame and benefit from smooth rendering on other clients, such as a character's eye angles or a carried object's position.

```csharp
[Sync( SyncFlags.Interpolate )] public Angles EyeAngles { get; set; }
```

Interpolation only affects how the value is displayed on remote clients - it does not change how or when data is sent.

# Collections

Sometimes you want to network collections such as an entire list or a dictionary. We provide special classes to do that.

```csharp
public enum AmmoCount
{
  Pistol,
  Rifle
}

public class MyComponent : Component 
{
  [Sync] public NetList<int> List { get; set; } = new();
  [Sync] public NetDictionary<AmmoCount,int> Dictionary { get; set; } = new();
}
```


You can initialize each in the declaration with `new()` or you can initialize the lists elsewhere, so long as you're doing so on the Owner of the network object. It doesn't matter if they are `null` for anyone else because they'll get created when they are networked if they need to be.


You can use `NetList<T>` and `NetDictionary<K,V>` like their regular counterparts. They contain indexers, `Add`, `Remove` and other methods you'd expect.

## Detecting Changes to Collections

The `[Change]` attribute does not fire when a collection's contents change, only when the property itself is reassigned. Instead, both `NetList<T>` and `NetDictionary<K,V>` expose an `OnChanged` event that fires whenever their contents change, on both the owner and all clients.

### NetList

```csharp
public class MyComponent : Component
{
    [Sync] public NetList<string> Players { get; set; } = new();

    protected override void OnStart()
    {
        Players.OnChanged += OnPlayersChanged;
    }

    private void OnPlayersChanged( NetListChangeEvent<string> e )
    {
        Log.Info( $"List changed: {e.Type} — New: {e.NewValue}, Old: {e.OldValue}, Index: {e.Index}" );
    }
}
```

The `NetListChangeEvent<T>` describes what happened:

| Property | Description |
|---|---|
| `Type` | The kind of change: `Add`, `Remove`, `Replace`, `Move`, or `Reset` |
| `Index` | The index where the change occurred |
| `MovedIndex` | The new index, only relevant for `Move` |
| `NewValue` | The value that was added or replaced with |
| `OldValue` | The value that was removed or replaced |

### NetDictionary

```csharp
public class MyComponent : Component
{
    [Sync] public NetDictionary<string, int> Scores { get; set; } = new();

    protected override void OnStart()
    {
        Scores.OnChanged += OnScoresChanged;
    }

    private void OnScoresChanged( NetDictionaryChangeEvent<string, int> e )
    {
        Log.Info( $"Scores changed: {e.Type} — Key: {e.Key}, New: {e.NewValue}, Old: {e.OldValue}" );
    }
}
```

The `NetDictionaryChangeEvent<TKey, TValue>` describes what happened:

| Property | Description |
|---|---|
| `Type` | The kind of change: `Add`, `Remove`, `Replace`, or `Reset` |
| `Key` | The key that was added, removed, or updated |
| `NewValue` | The new value for that key |
| `OldValue` | The previous value for that key |

:::info
`OnChanged` fires on **all clients**, not just the owner. This makes it a good place to update UI or trigger effects in response to collection changes, regardless of who made them.
:::


# Query Mode

By default the properties are automatically marked dirty when set, via codegen magic.. meaning that when you set a property, if it's different we'll send the updated value to everyone.


```csharp
[Sync]
public Vector3 Velocity
{
	get;
	set;
}
```

No Query Mode needed. The only way to change Velocity is via the setter, which when called will mark it as changed using invisible codegen magic on the setter.


```csharp
Vector3 _velocity;
[Sync]
public Vector3 Velocity
{
	get => _velocity;
	set => _velocity = value;
}
```

Again - no Query Mode needed. The only way we're setting _velocity is via the setter - so it can never get out of date.


```csharp
Vector3 _velocity;
[Sync( SyncFlags.Query )]
public Vector3 Velocity
{
	get => _velocity;
	set => _velocity = value;
}

void SetVelocity( Vector3 val)
{
    _velocity = val;
}
```

Query Mode needed - because when you call `SetVelocity` it changes `_velocity` and the network system doesn't know that the `Velocity` value has changed. This could be avoided in that case by setting `Velocity` instead of `_velocity`.


With `SyncFlags.Query`, the variable is instead checked for changes every network update, and sent if changed. This is marginally slower than non-query mode, but it means that you can sync special stuff like this.
