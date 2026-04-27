---
title: "Math Types"
icon: "📐"
created: 2025-06-16
updated: 2026-04-12
sources:
  - engine/Sandbox.System/Utility/MathX.cs
---

# Math Types

Learn the fundamental math types (`Vector3`, `Rotation`, `Transform`) required for moving, rotating, and scaling GameObjects in 3D space.

## Quick Working Example

```csharp
using Sandbox;

public sealed class MathExampleComponent : Component
{
	protected override void OnUpdate()
	{
		// 1. Vectors (Position and Direction)
		Vector3 forwardDirection = Vector3.Forward;
		
		// 2. Rotation (Orientation)
		Rotation spin = Rotation.FromYaw( 90f * Time.Delta );
		GameObject.LocalRotation *= spin;
		
		// 3. Transform (Position, Rotation, and Scale combined)
		Transform currentTransform = GameObject.WorldTransform;
		Vector3 worldForward = currentTransform.Forward;
		
		// Move the object forward
		GameObject.WorldPosition += worldForward * 100f * Time.Delta;
	}
}
```

### Working with Vectors

Vectors are used for positions, directions, velocities, and forces.

```csharp
// Creating Vectors
var a = new Vector3( 1, 2, 3 );
var b = new Vector3( 4, 5, 6 );

// Basic Math
var sum = a + b;
var scaled = a * 2.0f;

// Getting direction and distance
float distance = a.Distance( b );
Vector3 directionToB = (b - a).Normal; // .Normal makes the length exactly 1

// Comparing directions
float dotProduct = Vector3.Dot( a, b ); // 1 if parallel, 0 if perpendicular, -1 if opposite
```

### Working with Rotations

Always use the `Rotation` struct instead of `Angles` when doing math to avoid gimbal lock. Multiply rotations to combine them.

```csharp
// Create basic rotations
Rotation turnRight = Rotation.FromYaw( 90f );
Rotation lookUp = Rotation.FromPitch( 45f );

// Combine rotations by multiplying them
Rotation combined = turnRight * lookUp;

// Get directional vectors from a rotation
Vector3 forwardDir = combined.Forward;
Vector3 upDir = combined.Up;

// Smoothly interpolate between two rotations
Rotation current = GameObject.LocalRotation;
Rotation target = Rotation.LookAt( enemy.WorldPosition - GameObject.WorldPosition );
GameObject.LocalRotation = Rotation.Slerp( current, target, Time.Delta * 5f );
```

### Working with Transforms

A Transform wraps position, rotation, and scale. It is incredibly useful for converting local space (relative to the object) to world space (absolute).

```csharp
Transform t = GameObject.WorldTransform;

// Move a point from local space (e.g., 50 units in front of the object) to world space
Vector3 localOffset = new Vector3( 50, 0, 0 );
Vector3 absoluteWorldPosition = t.PointToWorld( localOffset );

// Move a world position into local space
Vector3 relativePosition = t.PointToLocal( absoluteWorldPosition );
```

### World vs Local

Every GameObject exposes both `WorldTransform`/`WorldPosition`/`WorldRotation` and `LocalTransform`/`LocalPosition`/`LocalRotation`. The difference is whether the transform is *relative to the parent* (Local) or *absolute in the scene* (World).

```csharp
// Top-level objects: Local and World are the same.
// Children of a parent: they differ.
GameObject.LocalPosition = new Vector3( 0, 0, 50 );  // 50 above the parent
GameObject.WorldPosition = new Vector3( 0, 0, 50 );  // 50 above the scene origin
```

The setter on `WorldPosition` recomputes the local transform behind the scenes so the object lands at the requested world point, regardless of where its parent is. Use `WorldPosition` when you have an absolute target (`MoveTo(enemy.WorldPosition)`); use `LocalPosition` when you're authoring an offset relative to the parent (`muzzle.LocalPosition = new Vector3(50, 0, 0)`).

## Core Types Reference

| Type | Description |
|---|---|
| `Vector3` | Represents 3D positions and directions (X, Y, Z). |
| `Vector2` | Represents 2D positions and directions (X, Y), useful for UI. |
| `Rotation` | Represents an orientation using Quaternions. Avoids Gimbal lock. |
| `Angles` | Represents an orientation using Euler angles (Pitch, Yaw, Roll). Easier for humans to read, but prone to Gimbal lock during calculations. |
| `Transform` | A struct containing `Vector3 Position`, `Rotation Rotation`, and `Vector3 Scale` (per-axis). |

## Troubleshooting

:::danger "My object is spinning wildly!"
When combining rotations, remember to **multiply** them (`rotA * rotB`), not add them (`rotA + rotB`). Adding Quaternions directly produces invalid rotations.
:::

:::warning "Angles vs Rotation"
If you find your object getting stuck when rotating or acting unpredictably when looking straight up or down, you might be suffering from Gimbal Lock caused by manipulating `Angles` directly. Try to perform your math using `Rotation` (Quaternions) instead.
:::

## Related Pages
- [Rotate an Object](../../how-to/rotate-object.md)
- [Transform Component](../../scene/transform.md)