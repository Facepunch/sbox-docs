---
title: Collisions and Triggers
sources:
  - engine/Sandbox.Engine/Scene/Components/Markers/ICollisionListener.cs
  - engine/Sandbox.Engine/Scene/Components/Markers/ITriggerListener.cs
  - engine/Sandbox.Engine/Systems/Physics/PhysicsWorld.cs
updated: 2026-04-25
created: 2026-04-27
---

# Collisions and Triggers

> **New to physics?** Physics overview is at **[Physics](index.md)**. This page is the reference for `ICollisionListener` and `ITriggerListener`.

React to physics events when objects bump into each other or enter defined zones using `ICollisionListener` and `ITriggerListener`.

## Quick Working Example

To detect when another physics object hits yours, implement the `Component.ICollisionListener` interface.

```csharp
using Sandbox;

public class DamageOnImpact : Component, Component.ICollisionListener
{
    public void OnCollisionStart( Collision collision )
    {
        Log.Info( $"We hit a GameObject!" );
    }
}
```

### Responding to Triggers

To detect when objects enter a trigger volume, implement `Component.ITriggerListener`. Remember that the Collider component on your trigger object must have the **Is Trigger** property checked.

```csharp
using Sandbox;

public class Spikes : Component, Component.ITriggerListener
{
    public void OnTriggerEnter( Collider other )
    {
        Log.Info( $"Something stepped on the spikes!" );
        
        var rb = other.GameObject.Components.Get<Rigidbody>();
        if ( rb != null )
        {
            rb.ApplyImpulse( Vector3.Up * 1000f );
        }
    }
}
```

## Configuration

The full `ICollisionListener` and `ITriggerListener` interfaces provide methods for start, ongoing, and stop events.

### ICollisionListener

| Method | Description |
|--------|-------------|
| `OnCollisionStart( Collision collision )` | Called on the first frame this object's collider touches another collider. |
| `OnCollisionUpdate( Collision collision )` | Called every physics step while the colliders remain touching. |
| `OnCollisionStop( CollisionStop collision )` | Called when the colliders stop touching. (`CollisionStop` has `Self` and `Other` but no `Contact`). |

### ITriggerListener

You can receive a `Collider` or `GameObject` parameter for triggers, optionally including the `self` trigger collider.

| Method | Description |
|--------|-------------|
| `OnTriggerEnter( Collider other )` | Called when a collider enters this trigger volume. |
| `OnTriggerExit( Collider other )` | Called when a collider exits this trigger volume. |
| `OnTriggerEnter( GameObject other )` | Called when a game object enters this trigger volume. |
| `OnTriggerExit( GameObject other )` | Called when a game object exits this trigger volume. |

### The Collision Struct

When a collision occurs, you receive a `Collision` struct. This gives you detailed data about the impact.

| Property | Description |
|----------|-------------|
| `collision.Self` | A `CollisionSource` containing the Collider and GameObject of *your* object. |
| `collision.Other` | A `CollisionSource` containing the Collider and GameObject of the *other* object. |
| `collision.Contact.Point` | The exact `Vector3` position in the world where the objects touched. |
| `collision.Contact.Normal` | A `Vector3` pointing away from the surface that was hit. |
| `collision.Contact.Speed` | How fast the objects collided as a `Vector3`. |

## Troubleshooting

:::danger
**My collision events aren't firing!**
Make sure both objects have a **Collider** component attached.
:::

:::warning
**Triggers are pushing the player away!**
Ensure your trigger zone's Collider component has the **Is Trigger** checkbox enabled. If it is unchecked, the physics engine will treat it as a solid wall.
:::

## Related Pages

- [Colliders](colliders.md)
- [Rigidbody](rigidbody.md)
