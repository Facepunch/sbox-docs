---
title: "Physics World"
icon: "🌍"
sources:
  - engine/Sandbox.Engine/Systems/Physics/PhysicsWorld.cs
  - engine/Sandbox.Engine/Scene/Scene/Scene.cs
updated: 2026-04-25
created: 2026-04-27
---

# Physics World

> **New to physics?** Physics overview is at **[Physics](index.md)**. This page is the reference for `PhysicsWorld`.

The `PhysicsWorld` is the central simulation space where all rigidbodies, colliders, and joints interact.

## Quick Working Example

```csharp
public class LowGravityZone : Component
{
	protected override void OnStart()
	{
		// Change the global gravity for the entire scene
		Scene.PhysicsWorld.Gravity = new Vector3( 0, 0, -200f );
		
		// Improve simulation accuracy at the cost of performance
		Scene.PhysicsWorld.SubSteps = 2;
	}
}
```

### Changing Gravity

You can dynamically change the gravity of the entire scene at runtime. This affects all `Rigidbody` components that have their `Gravity` property enabled.

```csharp
// Set gravity to point upwards
Scene.PhysicsWorld.Gravity = new Vector3( 0, 0, 800f );

// Disable gravity entirely
Scene.PhysicsWorld.Gravity = Vector3.Zero;
```

### Raycasting

The `PhysicsWorld` provides a `Trace` property that returns a `PhysicsTraceBuilder`. This is the underlying API used by `Scene.Trace`.

```csharp
// Fire a physics trace through the world
var trace = Scene.PhysicsWorld.Trace.Ray( startPos, endPos ).Run();

if ( trace.Hit )
{
	Log.Info( $"Hit: {trace.Body}" );
}
```

## Configuration

These properties exist on the `PhysicsWorld` instance, accessed via `Scene.PhysicsWorld`.

| Property | Description |
|:---|:---|
| `Gravity` | The global gravity vector. Default is typically `(0, 0, -800)`. |
| `AirDensity` | The density of the air, used for calculating air drag on objects. |
| `SleepingEnabled` | If true, idle bodies will go to sleep to save performance. |
| `SimulationMode` | The method used to simulate physics (e.g., `Continuous`, `Fixed`). |
| `SubSteps` | Breaks physics steps down into multiple smaller steps for higher accuracy. The default is 1. Higher values are more accurate but cost more performance. |

## Troubleshooting

:::danger Physics simulation running too slowly!
If you set `SubSteps` too high (e.g., 10+), the physics engine will simulate many times per frame. If your game has a high number of moving physics objects, this can easily tank your framerate.
:::

:::info Trace not returning GameObjects?
When you use `Scene.PhysicsWorld.Trace`, the result contains `PhysicsBody` and `PhysicsShape` references, which are the low-level physics objects. If you want high-level `GameObject` references, use `Scene.Trace` instead.
:::

## Related Pages
- [Rigidbody](rigidbody.md)
- [Colliders](colliders.md)
- [Tracing](../../scene/scenes/tracing.md)
