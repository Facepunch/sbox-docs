---
title: "Rigidbody"
icon: "🧊"
created: 2023-12-28
updated: 2026-04-11
sources:
  - engine/Sandbox.Engine/Scene/Components/Collider/Rigidbody.cs
---

# Rigidbody

> **New to physics?** Physics overview is at **[Physics](index.md)**. This page is the reference for `Rigidbody`.

The `Rigidbody` component puts a GameObject under the control of the physics engine, making it react to gravity, forces, and collisions.

## Quick Working Example

```csharp
public class JumpController : Component
{
    protected override void OnUpdate()
    {
        if ( Input.Pressed( "Jump" ) )
        {
            var rb = Components.Get<Rigidbody>();
            if ( rb != null )
            {
                // Apply a sudden upward burst of force (Impulse)
                rb.ApplyImpulse( Vector3.Up * 5000f );
            }
        }
    }
}
```

### Applying Forces

Forces are how you move Rigidbodies realistically.

```csharp
var rb = Components.Get<Rigidbody>();

// Add continuous force (like a thruster)
rb.ApplyForce( Vector3.Forward * 100f );

// Add instant force (like an explosion or jump)
rb.ApplyImpulse( Vector3.Up * 500f );

// Apply rotational force
rb.ApplyTorque( Vector3.Up * 100f );

// Clear all forces to stop it dead
rb.ClearForces();
```

### Sleeping and Waking

To improve performance, physics objects automatically go to "sleep" when they stop moving. You can manually force an object to sleep or wake up by toggling the `Sleeping` property.

```csharp
var rb = Components.Get<Rigidbody>();

// Force the rigidbody to sleep (stops simulating)
rb.Sleeping = true;

// Wake the rigidbody up (starts simulating again)
rb.Sleeping = false;
```

### Programmatic Movement (Kinematic/Proxy)

If you want to move an object smoothly through code while still interacting with the physics system (e.g., grabbing and dragging an object), use `SmoothMove` and `SmoothRotate`.

```csharp
var rb = Components.Get<Rigidbody>();

// Move towards the target position over 0.1 seconds
rb.SmoothMove( targetPosition, 0.1f, Time.Delta );

// Rotate towards the target rotation over 0.1 seconds
rb.SmoothRotate( targetRotation, 0.1f, Time.Delta );
```

## Configuration

| Property | Description |
|:---|:---|
| `Gravity` | Is gravity applied to this object? |
| `GravityScale` | Multiplier for gravity. 1.0 is normal, 0.0 is no gravity, 2.0 is double. |
| `Mass` | Calculated from colliders. You can check `OverrideMassCenter` or set `MassOverride` to manually control it. |
| `LinearDamping` | How quickly the object slows down its linear movement over time. |
| `AngularDamping` | How quickly the object slows down its rotation over time. |
| `Locking` | Allows you to lock movement or rotation on specific axes (e.g., locking Z rotation keeps an object upright). |
| `StartAsleep` | If true, the physics engine won't simulate this object until something hits it or applies a force. Good for performance. |
| `Velocity` | The current linear speed and direction of the object. |
| `AngularVelocity` | The current rotational speed of the object. |

## Visual Diagnostics

When working with Rigidbodies, you can visually debug their behavior in the Editor:

1. **Inspector Physics Display:** Select any GameObject with a Rigidbody. In the Inspector, you will see its current `Velocity` and `AngularVelocity` updating in real-time if the game is playing.
2. **Debug Overlay:** Use the `physics_debug` ConVar in the developer console to draw wireframes of the actual collision shapes and sleeping states. Sleeping objects often render in a different color to indicate they are not consuming CPU resources.

## Performance

*   **Avoid Every-Frame Modifictions:** Applying forces or impulses in a hot path like `OnUpdate` isn't inherently bad, but constantly reading and setting `Transform.Position` on a Rigidbody every frame forces the physics engine to recalculate spatial trees, severely degrading performance.
*   **StartAsleep:** If you are spawning hundreds of props in a level (e.g., debris or barrels), ensure `StartAsleep` is checked. This prevents the physics engine from simulating them until the player interacts with them.
*   **Kinematic vs Dynamic:** If an object only needs to move based on animation or code and push things out of the way, check the `MotionEnabled` property to make it Kinematic. This is much cheaper than a fully dynamic Rigidbody.

*   **Joint Interactions:** When a Rigidbody is attached to another object via a `Joint` component, applying massive impulses to one body can over-stress the joint constraint, causing unrealistic stretching or jittering. Tune the joint's breaking force or dampen the Rigidbody if this occurs.
*   **Parenting rigidbodies:** Rigidbodies nested under other Rigidbodies in the Scene Tree can cause conflicting physics resolutions. Generally, a Rigidbody should not be a child of another Rigidbody unless connected by a joint.

## Sample Projects

Download the **Walker Map** (`game/templates/walker.map`) or the [**Sweeper Sample**](../../build-games/samples-templates/sweeper.md) to experiment with Physics components. You can add a `Rigidbody` to a spawned object in these templates to immediately test gravity and collision behavior.

## Troubleshooting

:::danger "My Rigidbody falls straight through the floor!"
Make sure **both** the Rigidbody object and the floor have a Collider component attached. A Rigidbody without a Collider has no physical presence.
:::

:::warning "Setting Transform.Position instead of using forces"
If you manually set `GameObject.WorldPosition` every frame on a Rigidbody, you are teleporting it. This bypasses the physics system, meaning it won't collide properly or push other objects. If you need to drive it by code, use `SmoothMove`, or if you are building a character controller, look into custom kinematics or the `PlayerController`.
:::

## Related Pages
- [Colliders](colliders.md)
- [Player Controller](../../scene/components/reference/player-controller.md)
