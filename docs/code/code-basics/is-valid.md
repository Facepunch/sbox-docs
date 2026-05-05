---
title: "IsValid"
icon: "✅"
created: 2026-05-05
updated: 2026-05-05
---

# IsValid

Destroying a `GameObject` or `Component` doesn't erase the C# reference. Any variable that was pointing to it still holds a non-null object - it's just a dead one. Accessing its properties will do nothing useful or throw an exception.

This is why you can't rely on a plain `!= null` check:

```csharp
// ❌ Wrong - the reference is still non-null after Destroy()
if ( myObject != null )
{
    myObject.DoSomething(); // might throw an exception or do nothing
}

// ✅ Correct
if ( myObject.IsValid() )
{
    myObject.DoSomething();
}
```

## How It Works

The engine defines an `IValid` interface:

```csharp
public interface IValid
{
    bool IsValid { get; }
}
```

And a companion extension method that handles `null` *and* validity in one call:

```csharp
public static bool IsValid( this IValid? obj ) => obj != null && obj.IsValid;
```

So `myObject.IsValid()` is safe even if `myObject` is `null`. You do not need to check for `null` separately.

### What Each Type Checks

| Type | `IsValid` returns true when… |
|---|---|
| `GameObject` | Not destroyed and still in a scene |
| `Component` | Its `GameObject` is not null and still in a scene |
| `GameResource` (Model, Material, etc.) | Not explicitly destroyed |
| `SoundHandle` | The sound hasn't finished/been stopped |
| `Scene` | Its underlying scene world exists |

For example, `GameObject.IsValid` is literally:

```csharp
public virtual bool IsValid => !_destroyed && Scene is not null;
```

And `Component.IsValid`:

```csharp
public bool IsValid => GameObject is not null && Scene is not null;
```

## Common Patterns

### Checking before use

```csharp
void Update()
{
    if ( !_target.IsValid() ) return;

    var dist = WorldPosition.Distance( _target.WorldPosition );
}
```

### Filtering a list after a frame

```csharp
// Remove any destroyed targets from your list
_targets.RemoveAll( t => !t.IsValid() );
```

### Callbacks and delayed code

Async gaps and timers are common places where objects can be destroyed between frames:

```csharp
async Task ShootAfterDelay()
{
    await Task.DelaySeconds( 1.0f );

    // The object might have been destroyed during the delay
    if ( !this.IsValid() ) return;

    FireProjectile();
}
```

:::tip
`this.IsValid()` works on your own components too. It's a safe way to guard async callbacks after awaiting.
:::

## Implementing IValid in Your Own Class

If you have a class that can become invalid over time, implement `IValid`:

```csharp
public class SpawnMarker : IValid
{
    private bool _used;

    public bool IsValid => !_used;

    public void Use()
    {
        _used = true;
    }
}
```

Consumers can then use `.IsValid()` on it just like any engine type:

```csharp
var marker = GetNextMarker();

if ( marker.IsValid() )
{
    SpawnPlayerAt( marker );
    marker.Use();
}
```

:::info
Implementing `IValid` is optional for plain C# classes - it's most useful when your object has a concept of being "consumed", "released", or "expired" and you want callers to be able to check before using it.
:::
