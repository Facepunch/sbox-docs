---
title: "Apply Physics Forces"
icon: "💨"
sources:
  - engine/Sandbox.Engine/Scene/Components/Collider/Rigidbody.cs
updated: 2026-04-25
created: 2026-04-27
---

# Apply Physics Forces

> **New to physics?** Physics overview is at **[Physics](../systems/physics/index.md)**. This page is the recipe for one specific task.

Learn how to push, pull, or spin physics objects by applying forces, impulses, and torques.

## Quick Working Example

```csharp
using Sandbox;

public sealed class PushableBox : Component
{
	[RequireComponent] public Rigidbody Rigidbody { get; set; }

	protected override void OnUpdate()
	{
		if ( Input.Pressed( "attack1" ) )
		{
			// Apply a sudden burst of force upwards and forwards
			Rigidbody.ApplyImpulse( Vector3.Up * 300f + GameObject.WorldTransform.Forward * 500f );
		}
	}
}
```

### Applying Continuous Forces

If you are building something like a rocket thruster, a hovercraft, or an object caught in a windstorm, use `ApplyForce`. This applies a steady push to the Rigidbody.

```csharp
protected override void OnFixedUpdate()
{
	// Pushes the object forward continuously.
	// We use OnFixedUpdate because we are interacting with the physics system every frame.
	Rigidbody.ApplyForce( GameObject.WorldTransform.Forward * 2000f );
}
```

### Applying Instant Impulses

If you are building a jump mechanic, an explosion, or a gun that knocks objects back, use `ApplyImpulse`. This instantly adds velocity to the object.

```csharp
public void Jump()
{
	// Instantly shoot the object straight up
	Rigidbody.ApplyImpulse( Vector3.Up * 800f );
}
```

### Spinning Objects with Torque

To make an object spin without moving it positionally, use `ApplyTorque`.

```csharp
protected override void OnUpdate()
{
	if ( Input.Pressed( "Use" ) )
	{
		// Spin the object around the Z (Up) axis
		Rigidbody.ApplyTorque( Vector3.Up * 1000f );
	}
}
```

### Applying Forces at a Specific Position

By default, `ApplyForce` and `ApplyImpulse` apply energy to the object's center of mass. If you apply force to the *edge* of an object instead, it will cause the object to spin and move simultaneously—like shooting the edge of a signpost. 

You can do this using `ApplyForceAt` or `ApplyImpulseAt`, which take a `Vector3` position in world space.

```csharp
public void ShootBox( Vector3 hitPosition, Vector3 bulletDirection )
{
	// The box will fly backward and spin depending on exactly where the bullet hit it
	Rigidbody.ApplyImpulseAt( hitPosition, bulletDirection * 500f );
}
```

## Configuration

These methods are available directly on the `Rigidbody` component:

| Method | Description |
|---|---|
| `ApplyForce( Vector3 force )` | Applies continuous linear force to the center of mass. |
| `ApplyImpulse( Vector3 force )` | Applies an instant linear impulse to the center of mass. |
| `ApplyTorque( Vector3 force )` | Applies angular (rotational) force to the body. |
| `ApplyForceAt( Vector3 position, Vector3 force )` | Applies continuous linear force at a specific world position. |
| `ApplyImpulseAt( Vector3 position, Vector3 force )` | Applies an instant linear impulse at a specific world position. |
| `ClearForces()` | Clears all accumulated linear forces during this physics frame before they are simulated. |

## Troubleshooting

:::danger Object isn't moving!
If you call `ApplyForce` but the object doesn't move, ensure:
1. The `Rigidbody` has a `Collider` on it (or its children), otherwise it has no mass to push.
2. The `Rigidbody` is not set to `Kinematic` or locked on the axes you are trying to move.
3. The force magnitude is large enough. A force of `10f` on a heavy object will do nothing. Try multiplying your force by `1000f` to see if it starts moving.
:::

## Related Pages
* [Rigidbody](../systems/physics/rigidbody.md)
* [Collisions and Triggers](../systems/physics/collisions-and-triggers.md)
* [Trace / Raycast](trace-raycast.md) — find the impact point and surface normal to feed into `ApplyImpulseAt`.
