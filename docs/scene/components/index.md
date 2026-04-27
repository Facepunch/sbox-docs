---
title: "Components Overview"
icon: "🧩"
sources:
  - engine/Sandbox.Engine/Scene/Components/Component.cs
updated: 2026-04-25
created: 2026-04-27
---

# Components

A Component is a chunk of behaviour or data attached to a GameObject. It's where your code lives. Almost everything you'll do as a game dev — handle input, render a model, simulate physics, sync state across the network — is implemented as a component you write or one the engine ships.

```csharp
var rigidbody = GameObject.Components.Get<Rigidbody>();

var renderer = GameObject.Components.GetOrCreate<ModelRenderer>();
renderer.Model = Model.Load( "models/dev/box.vmdl" );
```

`Get<T>` returns the component if it's there, null if it isn't. `GetOrCreate<T>` adds one if it's missing. There's also `Create<T>` (always make a new one), `GetAll<T>` (return all matching components on this object and optionally its descendants), and the descendant/ancestor variants when you need to walk the tree.

## Common patterns for finding components {#getting-components}

When your component needs another component to function, prefer `[RequireComponent]`:

```csharp
public class MyPlayer : Component
{
    [RequireComponent] public Rigidbody Body { get; set; }
}
```

The engine ensures `Body` is non-null by the time your `OnAwake`/`OnStart` runs. If the GameObject doesn't have a `Rigidbody`, one is added automatically.

When you need to discover a component dynamically:

```csharp
// On this GameObject only
var rb = Components.Get<Rigidbody>();

// On this GameObject or any of its descendants
var renderer = Components.GetInDescendants<ModelRenderer>();

// Walking up the tree to find a parent component
var manager = Components.GetInAncestors<GameManager>();

// All components of a type, anywhere in the scene
foreach ( var camera in Scene.GetAllComponents<CameraComponent>() ) { ... }
```

`Scene.GetAllComponents<T>()` walks the whole scene tree — fine in `OnStart`, painful in `OnUpdate`. Cache references when you need them per-frame.

## Adding components from code

```csharp
var light = GameObject.Components.Create<PointLight>();
light.LightColor = Color.Red;
light.Radius = 500f;
```

The component runs through its full lifecycle (`OnAwake` → `OnEnabled` → `OnStart`) starting on the next tick.

## Where to go next

The four pages you'll actually need:

- [Component Lifecycle](component-lifecycle.md) — every override method (`OnAwake`, `OnStart`, `OnUpdate`, `OnFixedUpdate`, `OnEnabled`, `OnDisabled`, `OnDestroy`, `OnLoad`, `OnRefresh`, `OnValidate`, `OnPreRender`) and when each fires
- [Component Methods Reference](component-methods.md) — the full enumeration of overridable methods, including the less-common ones
- [Component Reference](reference/index.md) — every built-in component, organized alphabetically
- [Execution Order](execution-order.md) — for the rare case you need control over when your component runs relative to others

Less commonly:

- [Component Interfaces](component-interfaces/index.md) — `IDamageable`, `ITintable`, `IPressable`, etc. you can implement on your components
- [Component Events](events/index.md) — `IGameObjectNetworkEvents`, `IScenePhysicsEvents`, `IScenestartup` and friends, for hooking into engine-level events from a component
- [Component Versioning](component-versioning.md) — when a property's type or name changes and you need to migrate saved data

## Things that catch people out

**Component or GameObject disabled.** The most common "my component isn't running" cause. Both have an `Enabled` flag (the checkbox next to their name in the Inspector). Both must be true for `OnUpdate` to fire. A disabled parent also halts everything underneath it.

**Order between components is not guaranteed by default.** Two components with `OnUpdate` on the same frame run in a defined order, but that order isn't related to which one was added first or anything obvious. If you need explicit ordering, see [Execution Order](execution-order.md) or use a `GameObjectSystem` with stage control.

**`OnUpdate` doesn't run while paused.** `Time.Delta` becomes 0 when the game is paused, so `Time.Delta`-multiplied logic naturally pauses. But `OnUpdate` *itself* is also gated — if you need code to keep running even when paused (debug overlays, editor tooling), look at `Component.ExecuteInEditor` or `OnPreRender`.

## Related Pages

- [GameObject](../gameobject.md) — the container components attach to
- [Scene System](../index.md) — the bigger picture
- [Your First Component](../../tutorials/your-first-component.md) — line-by-line walkthrough if you've never written one
- [Create a Component (recipe)](../../how-to/create-a-component.md) — the minimum recipe
