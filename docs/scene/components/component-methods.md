---
title: "Component Methods Reference"
icon: "📋"
created: 2026-04-25
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Scene/Components/Component.cs
  - engine/Sandbox.Engine/Scene/Components/Component.Update.cs
  - engine/Sandbox.Engine/Scene/Components/Component.Loading.cs
  - engine/Sandbox.Engine/Scene/Components/Component.Network.cs
  - engine/Sandbox.Engine/Scene/Components/Component.Gizmos.cs
---

# Component Methods Reference

This is the **complete enumeration** of every overridable lifecycle and event method declared on the `Component` base class. The popular handful (`OnAwake`, `OnStart`, `OnUpdate`, `OnFixedUpdate`, `OnDestroy`) are covered in depth on the [Component Lifecycle](component-lifecycle.md) page; this reference exists to fill in everything else.

All methods listed here are `protected virtual` (or `public virtual`) on `Component` and have empty default bodies — you only override the ones you need.

## Quick Working Example

```csharp
using Sandbox;

public sealed class Bookkeeper : Component
{
    protected override void OnAwake()        => Log.Info( "memory allocated" );
    protected override void OnStart()        => Log.Info( "first tick about to run" );
    protected override void OnEnabled()      => Log.Info( "I'm now Active" );
    protected override void OnDisabled()     => Log.Info( "I'm no longer Active" );
    protected override void OnUpdate()       { /* every frame */ }
    protected override void OnFixedUpdate()  { /* every physics tick */ }
    protected override void OnPreRender()    { /* right before render, never on dedicated server */ }
    protected override void OnDestroy()      => Log.Info( "going away" );

    protected override Task OnLoad( LoadingContext context )
    {
        // Awaited by the loading screen before the scene starts.
        return Task.CompletedTask;
    }

    protected override void OnRefresh()      { /* after a network snapshot or hotload */ }
    protected override void OnValidate()     { /* property changed in the inspector */ }
    protected override void OnTagsChanged()  { /* GameObject.Tags was mutated */ }

    protected override void OnParentChanged( GameObject oldParent, GameObject newParent )
    {
        // We were re-parented in the hierarchy.
    }
}
```

## Initialisation & Teardown

| Method | Signature | When it fires |
|---|---|---|
| `OnAwake` | `protected virtual void OnAwake()` | Once per component, the first time the engine initialises it (driven by `InitializeComponent`). Other components on the same GameObject are not guaranteed to have woken yet — querying them is unsafe. |
| `OnStart` | `protected virtual void OnStart()` | Once, immediately before this component's first `OnUpdate` / `OnFixedUpdate`. By the time it runs, every component in the hierarchy has had `OnAwake` called, so component lookups are safe. |
| `OnEnabled` | `protected virtual void OnEnabled()` | Whenever the component transitions to `Active == true` (after Awake, when it or an ancestor GameObject is re-enabled). Symmetric with `OnDisabled`. |
| `OnDisabled` | `protected virtual void OnDisabled()` | Whenever the component transitions out of `Active`. Always pairs with a prior `OnEnabled`. Use it to unsubscribe from events you registered on enable. |
| `OnDestroy` | `protected virtual void OnDestroy()` | Once, when the component or its GameObject is destroyed. After this returns, `GameObject` is set to `null` and `IsValid` is `false`. |
| `OnParentDestroy` | `public virtual void OnParentDestroy()` | The parent GameObject is being destroyed. The default behaviour is to be destroyed along with the parent — override this if you want to detach to a different parent first. |

## Per-Frame & Per-Tick

| Method | Signature | When it fires |
|---|---|---|
| `OnUpdate` | `protected virtual void OnUpdate()` | Every rendered frame, while the component is `Active` and `ShouldExecute`. Multiply by `Time.Delta` for frame-rate-independent logic. |
| `OnFixedUpdate` | `protected virtual void OnFixedUpdate()` | Every fixed timestep, paced by the scene's physics tick rate. `Time.Delta` here is the fixed interval. Does not run while the scene is paused. |
| `OnPreRender` | `protected virtual void OnPreRender()` | Every frame, just before the scene is rendered. **Never called on a dedicated server.** Useful for last-minute visual tweaks (camera pose, bone overrides, screen-space effects). |

`OnFixedUpdate` runs *before* the physics step. If you need a hook that runs after `OnFixedUpdate` but immediately before the integrator (or right after), implement [`IScenePhysicsEvents`](../../systems/physics/physics-events.md) instead.

## Editor & Hot-Reload

| Method | Signature | When it fires |
|---|---|---|
| `OnValidate` | `protected virtual void OnValidate()` | Immediately after deserialisation, and when a `[Property]` is edited in the inspector. The engine wraps the call in a try/catch so a thrown exception is logged rather than fatal. |
| `OnRefresh` | `protected virtual void OnRefresh()` | Called after the component has been refreshed — most commonly after a network snapshot is applied, or after code hotload restores state into a new instance. |
| `Reset` | `public virtual void Reset()` | The user clicked "Reset" in the inspector. The default implementation walks the type's `[Property]` fields/properties and resets each to its default value via `SerializedObject`. Override to add custom reset logic, but call `base.Reset()` if you still want defaults applied. |
| `DrawGizmos` | `protected virtual void DrawGizmos()` | Called in the editor while the component is selected, so you can draw debug shapes. The engine catches and logs exceptions. |

## Async Loading

| Method | Signature | When it fires |
|---|---|---|
| `OnLoad()` | `protected virtual Task OnLoad()` | Called once during the scene's loading phase. Return a non-completed `Task` and the loading screen will wait for it before the scene starts. |
| `OnLoad(LoadingContext)` | `protected virtual Task OnLoad( LoadingContext context )` | Overload that receives a `LoadingContext`. Default implementation forwards to the parameterless `OnLoad()`. Override the contextful overload if you need to report progress or cooperate with the loader. |

The engine schedules loading via `LaunchLoader` which calls `OnLoad`, and if the returned task is not yet complete, registers it as a scene loading task and tags the GameObject with `GameObjectFlags.Loading` until it finishes.

## Tags & Hierarchy

| Method | Signature | When it fires |
|---|---|---|
| `OnTagsChanged` | `protected virtual void OnTagsChanged()` | The `GameObject.Tags` set was mutated (added, removed, etc.). The component receives this *after* the change is applied. |
| `OnParentChanged` | `protected virtual void OnParentChanged( GameObject oldParent, GameObject newParent )` | The owning GameObject was re-parented. Both arguments may be `null` (e.g. when initially attached, `oldParent` is `null`). |
| `OnParentDestroy` | `public virtual void OnParentDestroy()` | See [Initialisation & Teardown](#initialisation--teardown). |

## Networking-Adjacent

`Component.Network.cs` does not add new overridable lifecycle methods — but it adds infrastructure that overrides care about. Useful members:

- `Network` — exposes the `GameObject.NetworkAccessor` for ownership checks, RPC invocation, etc.
- `IsProxy` — `true` when this object is owned by another client. Most authoritative gameplay code starts with `if ( IsProxy ) return;`.

For network ownership transitions, implement [`IGameObjectNetworkEvents`](events/igameobjectnetworkevents.md) — that's the canonical hook, not a method on `Component`.

## Obsolete

| Method | Notes |
|---|---|
| `OnDirty` | Marked `[Obsolete]`. Was previously paired with `[MakeDirty]` to receive a callback after a property setter ran. New code should use `[Property]` change notifications or explicit invalidation logic instead. |
| `EditLog( string name, object source )` | `[Obsolete]`. Replaced by `Scene.Editor.UndoScope` / `Scene.Editor.AddUndo`. |

## Scene Event Interfaces (Not Methods on Component)

Several reactions to engine events are **not** `Component` virtuals — they are interfaces you opt into by implementing on your component class. The engine dispatches them via the scene event bus the same way it would for a `GameObjectSystem`. The full list of built-ins lives in [Component Events](events/index.md); the most commonly used:

| Interface | What it gives you |
|---|---|
| `IScenePhysicsEvents` | `PrePhysicsStep` / `PostPhysicsStep` / `OnFellAsleep` / `OnOutOfBounds`. See [Physics Events](../../systems/physics/physics-events.md). |
| `ISceneCollisionEvents` | Global collision callbacks for the whole scene. For per-object collisions use `Component.ICollisionListener` instead. |
| `ISceneLoadingEvents` | `BeforeLoad` / `OnLoad` / `AfterLoad`, scene-wide hooks parallel to per-component `Component.OnLoad`. |
| [`ISceneStartup`](events/iscenestartup.md) | `OnHostPreInitialize`, `OnHostInitialize`, `OnClientInitialize` — fired once per scene start. |
| [`IGameObjectNetworkEvents`](events/igameobjectnetworkevents.md) | `NetworkOwnerChanged`, `StartControl`, `StopControl` — targeted to the GameObject that changed. |
| `Component.INetworkListener` | Host-only hooks for connection lifecycle (`OnConnected`, `OnActive`, `AcceptConnection`, `OnBecameHost`, `OnDisconnected`). Used by [Network Helper](../../systems/networking-multiplayer/network-helper.md). |

Implementing one of these on a component costs nothing if you don't override its methods — every method has a default empty body.

## Related Pages

- [Component Lifecycle](component-lifecycle.md) — narrative walkthrough of the popular methods.
- [Component Events](events/index.md) — the scene-event interfaces in depth.
- [Execution Order](execution-order.md) — dispatch order across components in a scene.
- [Physics Events](../../systems/physics/physics-events.md) — `IScenePhysicsEvents` reference.
