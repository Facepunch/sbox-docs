---
title: "Colliders"
icon: "📐"
sources:
  - engine/Sandbox.Engine/Scene/Components/Collider/Collider.cs
  - engine/Sandbox.Engine/Scene/Components/Collider/BoxCollider.cs
  - engine/Sandbox.Engine/Scene/Components/Collider/SphereCollider.cs
  - engine/Sandbox.Engine/Scene/Components/Collider/CapsuleCollider.cs
  - engine/Sandbox.Engine/Scene/Components/Collider/ModelCollider.cs
  - engine/Sandbox.Engine/Scene/Components/Markers/ITriggerListener.cs
  - engine/Sandbox.Engine/Scene/Components/Markers/ICollisionListener.cs
updated: 2026-04-25
created: 2026-04-27
---

# Colliders

> **New to physics?** Physics overview is at **[Physics](index.md)**. This page is the reference for collider components.

Colliders define the invisible physical shape of a GameObject used for collision detection and physics simulation.

## Quick Working Example

```csharp
public class BouncePad : Component
{
	protected override void OnStart()
	{
		// Add a BoxCollider to this GameObject
		var box = Components.GetOrCreate<BoxCollider>();
		
		// Configure its dimensions and physical properties
		box.Scale = new Vector3( 100, 100, 10 );
		box.Elasticity = 1.0f; // Maximum bounce (0..1)
		box.Friction = 0.5f;
	}
}
```

## Types of Colliders

s&box provides several primitive and complex collider types. Primitive shapes are much cheaper to simulate than complex models.

*   `BoxCollider`: A basic cube or rectangle.
*   `SphereCollider`: A perfect sphere. Very cheap to simulate.
*   `CapsuleCollider`: A cylinder with rounded ends, perfect for characters.
*   `HullCollider`: A custom convex shape.
*   `ModelCollider`: Uses the physics hull embedded inside a 3D `Model` asset.

### Setting up a Trigger Volume

If you want an invisible area that detects when something enters it without physically blocking it (like a laser tripwire or a healing zone), you set the collider as a Trigger.

```csharp
public class HealingZone : Component
{
	protected override void OnStart()
	{
		var sphere = Components.GetOrCreate<SphereCollider>();
		sphere.Radius = 250f;
		
		// Make it non-solid
		sphere.IsTrigger = true;
		
		// Use delegates to listen to events directly
		sphere.OnObjectTriggerEnter = ( GameObject obj ) =>
		{
			Log.Info( $"Started healing {obj.Name}" );
		};
	}
}
```

Alternatively, you can implement `ITriggerListener` on any component attached to the same GameObject. See [Collisions and Triggers](collisions-and-triggers.md) for more details.

### Making a Conveyor Belt

You can push objects that rest on a collider without using code by setting its `SurfaceVelocity`.

```csharp
var collider = Components.Get<BoxCollider>();

// Push objects 200 units per second along the X axis
collider.SurfaceVelocity = new Vector3( 200f, 0, 0 );
```

## Configuration

These properties exist on the base `Collider` component and apply to all shapes.

| Property | Description |
|:---|:---|
| `Static` | If true, this collider will not move and ignores physical forces (like floors/walls). |
| `IsTrigger` | If true, objects pass through it, but it fires trigger events. |
| `Friction` | How slippery the surface is (0 to 1). |
| `Elasticity` | How bouncy the surface is (0 = no bounce, 1 = perfectly bouncy). |
| `Surface` | The surface type (e.g., wood, metal) which determines impact sounds and decals. |
| `SurfaceVelocity` | Moves objects resting on this collider as if it were a moving surface. |
| `OnTriggerEnter` / `Exit` | Delegate callbacks fired when another collider enters or leaves this trigger. |
| `OnObjectTriggerEnter` / `Exit` | Delegate callbacks fired when a GameObject enters or leaves this trigger. |

## Troubleshooting

:::danger "My player walks right through the wall!"
1. Verify the wall has a `Collider` component attached.
2. Ensure the `Collider` on the wall does **not** have `IsTrigger` checked.
3. Check that the player character has a `Collider` or `CharacterController` attached.
:::

:::warning "Triggers are pushing the player away!"
If you intended to create a non-solid volume to detect players (like a zone or a checkpoint), ensure your `Collider` component has the **Is Trigger** checkbox enabled. If it is unchecked, the physics engine will treat it as a solid wall.
:::

## Related Pages
- [Rigidbody](rigidbody.md)
- [Collisions and Triggers](collisions-and-triggers.md)
- [Tracing](../../scene/scenes/tracing.md)
