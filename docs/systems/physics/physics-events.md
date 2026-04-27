---
title: "Physics Events"
icon: "🚪"
created: 2024-10-30
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Scene/Events/IScenePhysicsEvents.cs
  - engine/Sandbox.Engine/Scene/Components/Collider/ModelPhysics.cs
  - engine/Sandbox.Engine/Scene/Components/Collider/Rigidbody.cs
---

# Physics Events

> **New to physics?** Physics overview is at **[Physics](index.md)**. This page is the reference for `IScenePhysicsEvents`.

The `IScenePhysicsEvents` interface lets a component hook logic into the physics step. This is useful when an object or system needs to read or write to the physics world at a precise point in the simulation tick — for example, applying forces just before the integrator runs, or sampling positions immediately after.

## Quick Working Example

```csharp
using Sandbox;

public sealed class PhysicsHookComponent : Component, IScenePhysicsEvents
{
    void IScenePhysicsEvents.PrePhysicsStep()
    {
        // Runs right after FixedUpdate, before the engine integrates physics
        // Apply forces, set velocities, etc.
    }

    void IScenePhysicsEvents.PostPhysicsStep()
    {
        // Runs after the integrator has run
        // Sample post-integration positions, clamp velocities, etc.
    }
}
```

## Interface

```csharp
public interface IScenePhysicsEvents : ISceneEvent<IScenePhysicsEvents>
{
    /// <summary>
    /// Called before the physics step is run.
    /// This is called pretty much right after FixedUpdate.
    /// </summary>
    void PrePhysicsStep() { }

    /// <summary>
    /// Called after the physics step is run.
    /// </summary>
    void PostPhysicsStep() { }
}
```

Both methods have empty default implementations, so you only need to override the one(s) you need.

## When to Use

The `OnFixedUpdate` callback runs *before* the physics step. `IScenePhysicsEvents.PrePhysicsStep` runs in the same window — typically right after `OnFixedUpdate` — but is part of the scene event bus, so it can be implemented by components, gameobject systems, or other scene-event subscribers without the lifecycle overhead of a regular component callback.

Use `IScenePhysicsEvents` when:

- You're building a gameplay system that needs to participate in the physics tick without being a full `Component` (e.g., a `GameObjectSystem`).
- You want both pre- and post-step hooks symmetrically.
- Your logic needs to coordinate with `Rigidbody` or `ModelPhysics`, both of which already implement this interface internally.

For most gameplay code, prefer `OnFixedUpdate` — it's simpler and runs at the same point as `PrePhysicsStep`.

- **Order of execution.** All `IScenePhysicsEvents` subscribers are dispatched together; do not assume a particular ordering between subscribers without explicit prioritization.
- **Variable physics tick rate.** The physics step runs at a fixed interval (typically 50 Hz), but the *number* of physics steps per render frame can vary if the game falls behind. `PrePhysicsStep` and `PostPhysicsStep` will fire once per actual physics step, not once per render frame.
- **Disabled components.** A disabled component does not receive scene events, including `PrePhysicsStep` / `PostPhysicsStep`.

## Related Pages

- [Physics World](physics-world.md)
- [Rigidbody](rigidbody.md)
- [Component Lifecycle](../../scene/components/component-lifecycle.md) — for `OnFixedUpdate` and other lifecycle hooks
