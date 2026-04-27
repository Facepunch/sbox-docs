---
title: "Component Lifecycle"
icon: "⏱️"
created: 2023-12-28
updated: 2025-06-25
sources:
  - engine/Sandbox.Engine/Scene/Components/Component.cs
  - engine/Sandbox.Engine/Scene/Components/Component.Update.cs
  - engine/Sandbox.Engine/Scene/Components/Component.Loading.cs
---

# Component Lifecycle

When you write a component, the engine calls a sequence of methods on it: `OnAwake` when it's created, `OnStart` when everything's wired up, `OnUpdate` every frame, `OnDestroy` when it's gone. Knowing which method runs when is most of what makes your code work or not work.

This page covers the methods you'll override 90% of the time. The full enumeration (including the niche ones like `OnTagsChanged`, `OnParentChanged`, `OnPreRender`) is in the [Component Methods Reference](component-methods.md).

## A demonstration component

Drop this on any GameObject and watch the console:

```csharp
using Sandbox;

public sealed class LifecycleDemo : Component
{
    protected override void OnAwake()    => Log.Info( "1. OnAwake" );
    protected override void OnStart()    => Log.Info( "2. OnStart" );
    protected override void OnEnabled()  => Log.Info( "   OnEnabled" );
    protected override void OnDisabled() => Log.Info( "   OnDisabled" );
    protected override void OnDestroy()  => Log.Info( "3. OnDestroy (eventually)" );

    protected override void OnUpdate()
    {
        // Tiny rotation each frame so you know it's ticking
        GameObject.LocalRotation *= Rotation.FromYaw( 90f * Time.Delta );
    }
}
```

The order on a fresh spawn: `OnAwake` → `OnEnabled` → `OnStart`, then `OnUpdate` every frame, until you destroy it (`OnDisabled` then `OnDestroy`).

## When each runs

`OnAwake` runs the moment the component is created. **Other components on the same GameObject may not have awoken yet.** Don't query siblings here.

`OnStart` runs after every component on the GameObject has finished `OnAwake`. This is where you cache references to siblings.

```csharp
private Rigidbody _rb;

protected override void OnStart()
{
    _rb = Components.Get<Rigidbody>();   // safe — it's awake by now
}
```

`OnUpdate` runs every rendered frame. Multiply continuous changes by `Time.Delta` so they're framerate-independent. Cache anything you'd otherwise look up per frame.

`OnFixedUpdate` runs at a fixed rate (typically 50 Hz) regardless of framerate. This is where physics-driving code goes — `ApplyForce`, `ApplyImpulse`, anything that needs to behave consistently across machines.

```csharp
protected override void OnFixedUpdate()
{
    if ( _rb is not null ) _rb.ApplyForce( Vector3.Up * 500f );
}
```

`OnEnabled` and `OnDisabled` fire whenever the component flips between enabled and disabled. Use them to subscribe to and unsubscribe from external events:

```csharp
protected override void OnEnabled()  => SomeStaticEvent.Listen += React;
protected override void OnDisabled() => SomeStaticEvent.Listen -= React;
```

A *disabled GameObject* disables all its components — `OnDisabled` fires on each. A *destroyed GameObject* runs `OnDestroy` on each.

## Hot reload doesn't restart the lifecycle

When you save a `.cs` file, the engine recompiles and migrates your live state into the new assembly. **`OnAwake` and `OnStart` do not re-run.** If you have setup that needs to re-derive after a hotload (rebuild a cache, re-subscribe to something), put it in `OnRefresh`:

```csharp
protected override void OnRefresh()
{
    RebuildExpensiveCache();
}
```

`OnRefresh` also fires after deserialization (loading a saved scene), which is convenient — same setup runs at boot and on hotload.

## Networked games: check `IsProxy`

The lifecycle runs on every peer. If you're writing logic that should only run on the owner of a networked GameObject (e.g., handling player input), gate it on `IsProxy`:

```csharp
protected override void OnUpdate()
{
    if ( IsProxy ) return;     // someone else owns this object
    HandleLocalInput();
}
```

Without that guard, every client tries to drive the same object and the snapshot keeps overwriting them, causing rubber-banding.

## Pausing

When the scene is paused (`Scene.TimeScale = 0`):

- `Time.Delta` is 0, so anything multiplied by it naturally pauses.
- `OnFixedUpdate` does not fire — physics is suspended.
- `OnUpdate` still fires by default. UI, debug overlays, menu logic continue to work.

If you have a component that should pause too, gate its work on `Scene.TimeScale > 0` or implement `Component.ShouldExecute` to return false when paused.

## Cheat sheet

| Method | When it fires | Use it for |
|---|---|---|
| `OnAwake` | Component created | Internal setup (don't touch siblings yet) |
| `OnStart` | All sibling awakes done | Cache sibling references, do work that depends on them |
| `OnEnabled` | Component goes enabled | Subscribe to external events |
| `OnDisabled` | Component goes disabled | Unsubscribe |
| `OnUpdate` | Every rendered frame | Game logic, input, visuals (multiply by `Time.Delta`) |
| `OnFixedUpdate` | Every physics tick (~50 Hz) | Forces, character movement, anything tied to physics |
| `OnDestroy` | Object/component destroyed | Release external resources |
| `OnRefresh` | After hotload or deserialize | Rebuild caches, re-subscribe |
| `OnValidate` | Property changed in editor | Clamp values, validate inputs |
| `OnLoad` | Awaited during scene load | Load assets, generate procedural data — the loading screen waits for it |

## Common mistakes

**`OnAwake` queries another component → null.** Move the query to `OnStart`.

**Visual movement in `OnFixedUpdate`.** Physics ticks 50 Hz; render runs at framerate. Visuals updated only at 50 Hz look choppy on high-refresh monitors. Keep `OnFixedUpdate` for forces; do visual logic in `OnUpdate`.

**`Components.GetAll<T>()` in `OnUpdate`.** Walks the tree every call. Cache in `OnStart`.

**Forgetting to unsubscribe.** Subscribe in `OnEnabled` → unsubscribe in `OnDisabled`. If you only subscribe in `OnStart` and never unsubscribe, your component keeps receiving events after destruction and crashes.

## Related Pages

- [Component Methods Reference](component-methods.md) — every overridable method, including `OnPreRender`, `OnTagsChanged`, `OnParentChanged`, etc.
- [Execution Order](execution-order.md) — controlling order between sibling components
- [Use Async Tasks](../../how-to/component-async.md) — async work without blocking the lifecycle
- [Hotloading](../../code/code-basics/hotloading.md) — what hotload preserves and what it doesn't
