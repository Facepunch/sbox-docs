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


You do not need to check for `null` separately. `myObject.IsValid()` is safe even if `myObject` is `null`.

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
