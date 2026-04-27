---
title: "IGameObjectNetworkEvents"
icon: "🔄"
created: 2026-04-25
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Scene/Events/IGameObjectNetworkEvents.cs
---

# IGameObjectNetworkEvents

`IGameObjectNetworkEvents` is a **targeted** scene-event interface — it fires only on the GameObject whose network ownership or control state is changing, not the whole scene. Implement it on a component to react when this object becomes locally controlled, becomes a proxy, or has its owner changed.

## Quick Working Example

```csharp
using Sandbox;

public sealed class Vehicle : Component, IGameObjectNetworkEvents
{
    [Property] public CameraComponent DriverCamera { get; set; }

    void IGameObjectNetworkEvents.StartControl()
    {
        // We are now the controller (no longer a proxy).
        if ( DriverCamera.IsValid() )
            DriverCamera.Enabled = true;
    }

    void IGameObjectNetworkEvents.StopControl()
    {
        // Someone else owns this now, or we dropped ownership.
        if ( DriverCamera.IsValid() )
            DriverCamera.Enabled = false;
    }

    void IGameObjectNetworkEvents.NetworkOwnerChanged( Connection newOwner, Connection previousOwner )
    {
        Log.Info( $"{GameObject.Name} owner: {previousOwner?.DisplayName ?? "<none>"} -> {newOwner?.DisplayName ?? "<none>"}" );
    }
}
```

## Interface

```csharp
public interface IGameObjectNetworkEvents : ISceneEvent<IGameObjectNetworkEvents>
{
    /// <summary>Called before we are about to drop ownership of a network GameObject.</summary>
    internal void BeforeDropOwnership() { }

    /// <summary>Called when the owner of a network GameObject is changed.</summary>
    void NetworkOwnerChanged( Connection newOwner, Connection previousOwner ) { }

    /// <summary>We have become the controller of this object, we are no longer a proxy.</summary>
    void StartControl() { }

    /// <summary>This object has become a proxy, controlled by someone else.</summary>
    void StopControl() { }
}
```

`BeforeDropOwnership` is `internal` and not callable from user code — it is only here for the engine's own teardown path. Treat the public surface as `NetworkOwnerChanged`, `StartControl`, and `StopControl`.

All three methods have empty default implementations, so a component implementing the interface only needs to override the ones it cares about.

## When Each Method Fires

| Method | Fires when | Typical use |
|---|---|---|
| `NetworkOwnerChanged( newOwner, previousOwner )` | The owning `Connection` for this GameObject changes — including initial assignment from `null` and clearing back to `null`. | Logging, swapping nameplates, recolouring per-player. |
| `StartControl()` | The local machine *gains* control: `IsProxy` transitions from `true` to `false`. | Enabling input handlers, switching to a first-person camera, claiming UI. |
| `StopControl()` | The local machine *loses* control: `IsProxy` transitions from `false` to `true`. | Disabling input, releasing the camera, switching to a remote-puppet animation graph. |

`StartControl` / `StopControl` are about **local control**, not ownership in the abstract: each machine fires the pair that matches its own perspective. The owner machine sees `StartControl` once when it gains control; every other machine sees `StopControl` (or simply never sees `StartControl`) for that same object.

## Targeted Dispatch

Unlike most `ISceneEvent<T>`-derived interfaces, `IGameObjectNetworkEvents` is dispatched **only** to components on the specific GameObject whose ownership state is changing. You will not receive a callback for a different vehicle's ownership change — you'd have to subscribe to that vehicle's components directly, or use a global event of your own.

This makes it cheap and safe to put on every networked GameObject: there is no per-frame cost, and you don't have to filter by `GameObject` in the body.

- **Initial spawn.** When a networked GameObject is first spawned, `NetworkOwnerChanged` fires with `previousOwner == null` and `newOwner` set to whoever spawned it. The owning client also receives `StartControl` at the same time; remote clients do not receive `StartControl`.
- **Proxy → proxy.** If ownership transfers between two *remote* clients, your local machine sees `NetworkOwnerChanged` but neither `StartControl` nor `StopControl` — your control state hasn't changed.
- **Disabled components don't receive events.** A component must be `Active` to receive scene events. If you toggle the component off in `StopControl` and rely on `StartControl` to turn it back on, you'll never receive `StartControl` because the disabled component cannot listen. Toggle child GameObjects or sibling components instead.
- **Combine with `IsProxy`.** During `OnUpdate`/`OnFixedUpdate` the canonical authority check is still `if ( IsProxy ) return;`. `StartControl` / `StopControl` are *transitions* — useful for one-shot setup and teardown, not steady-state authority gating.

## Related Pages

- [Component Events](index.md)
- [Ownership](../../../systems/networking-multiplayer/ownership.md)
- [Networked Objects](../../../systems/networking-multiplayer/networked-objects.md)
- [Component Methods Reference](../component-methods.md)
- [ISceneStartup](iscenestartup.md)
