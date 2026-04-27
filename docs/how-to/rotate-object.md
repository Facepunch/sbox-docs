---
title: "Rotate an Object"
icon: "🔃"
sources:
  - engine/Sandbox.Engine/Scene/GameObject/GameObject.cs
created: 2026-04-27
updated: 2026-04-27
---

# Rotate an Object

How to continuously rotate a GameObject, set its absolute rotation, or make it look at a target.

## Quick Working Example

```csharp
using Sandbox;

public sealed class Spinner : Component
{
    [Property] public float SpinSpeed { get; set; } = 90f; // Degrees per second

    protected override void OnUpdate()
    {
        // Continuously rotate the GameObject around its local Z (Up) axis
        GameObject.LocalRotation *= Rotation.FromYaw( SpinSpeed * Time.Delta );
    }
}
```

### Setting an Absolute Rotation
If you want to snap an object to a specific angle (e.g., exactly 90 degrees on the Y axis), you can set its `WorldRotation` or `LocalRotation` directly.

```csharp
// Pitch (Y axis), Yaw (Z axis), Roll (X axis)
GameObject.LocalRotation = Rotation.From( 0, 90, 0 );
```

### Looking at a Target
Often you want an object (like a turret, a camera, or a billboard) to point directly at another object. Use `Rotation.LookAt()`.

```csharp
public class LookAtPlayer : Component
{
    [Property] public GameObject Target { get; set; }

    protected override void OnUpdate()
    {
        if ( Target == null ) return;

        // Calculate the direction from us to the target
        Vector3 direction = Target.WorldPosition - GameObject.WorldPosition;
        
        // Point the GameObject's Forward axis towards the target
        GameObject.WorldRotation = Rotation.LookAt( direction, Vector3.Up );
    }
}
```

### Smoothly Rotating Towards a Target
If you don't want the object to snap instantly to look at the target, you can use `Rotation.Slerp()` (Spherical Linear Interpolation) to smoothly turn over time.

```csharp
protected override void OnUpdate()
{
    if ( Target == null ) return;

    Vector3 direction = Target.WorldPosition - GameObject.WorldPosition;
    Rotation targetRotation = Rotation.LookAt( direction, Vector3.Up );

    // Smoothly turn from our current rotation to the target rotation
    float turnSpeed = 5.0f;
    GameObject.WorldRotation = Rotation.Slerp( GameObject.WorldRotation, targetRotation, Time.Delta * turnSpeed );
}
```

## Troubleshooting

:::danger "My Rigidbody is acting weird when I rotate it!"
If your GameObject has a `Rigidbody` component, directly setting `GameObject.WorldRotation` counts as a "teleport". The physics engine doesn't like this and the object won't interact properly with other physics objects during that frame.

If you need to rotate a Rigidbody smoothly, use `Rigidbody.SmoothRotate()`, or apply torque (`Rigidbody.ApplyTorque`).
:::

:::warning "Can I just edit X, Y, and Z angles directly?"
Because s&box uses Quaternions, there are no `Transform.Rotation.x += 1` properties. You must always use the `Rotation` struct and its helper methods like `Rotation.FromPitch()`, `Rotation.FromYaw()`, or `Rotation.From()`.
:::
