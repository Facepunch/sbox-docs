---
title: Sweeper Sample
sources:
  - game/samples/sweeper
updated: 2026-04-25
created: 2026-04-27
---

# Sweeper Sample

The `sweeper` sample, located at `game/samples/sweeper/`, is a basic demonstration project. Despite its name historically associating with "Minesweeper", this sample currently provides a foundational 3D movement script (`CubeController.cs`) to show how to process input and move a GameObject.

## Quick Working Example

You can explore this sample by opening its provided scene and pressing Play. The primary logic is driven by the `CubeController` component, which demonstrates analog input reading:

```csharp
using Sandbox;

/// <summary>
/// An example component. This moves the object its attached to based on inputs from the client.
/// </summary>
public sealed class CubeController : Component
{
	/// <summary>
	/// This can be configured in the scene!
	/// </summary>
	[Property]
	public float MoveSpeed { get; set; } = 100.0f;

	/// <summary>
	/// This is called every frame
	/// </summary>
	protected override void OnUpdate()
	{
		// get the current rotation as pitch yaw roll angles
		var angles = WorldRotation.Angles();

		// add the pitch and yaw from the input
		angles += Input.AnalogLook;

		// zero the pitch and roll - because we just want it to yaw
		angles.pitch = 0;
		angles.roll = 0;

		// set the new rotation
		WorldRotation = angles;

		// Create a vector3 to hold our move direction
		var moveDirection = Vector3.Zero;

		// Move in the direction of our input. AnalogMove gets built automatically from
		// "forward" "left" etc buttons. It also grabs the controller stick input
		// Multiply this by the configured move speed and the time delta to make it frame rate independent
		moveDirection += Input.AnalogMove * MoveSpeed * Time.Delta;

		// Multiply the movement direction by our rotation so we move in the direction we're facing, and
		// add it to our current position.
		WorldPosition += WorldRotation * moveDirection;
	}
}
```

## What it Demonstrates

This sample is an excellent learning resource for developers new to the engine, as it demonstrates:

1. **Input System Usage:** How to read `Input.AnalogLook` (mouse delta/right stick) and `Input.AnalogMove` (WASD/left stick) in a single unified vector.
2. **Transform Math:** Multiplying a movement vector (`moveDirection`) by a `Rotation` to ensure the object moves *forward* relative to where it is currently facing, rather than globally forward.
3. **Framerate Independence:** Always multiplying continuous movement offsets by `Time.Delta`.

## Common Patterns

1. **Direct Transform Control:** For simple arcade games, floating cameras, or kinematic characters, manipulating `WorldPosition` directly is often easier than configuring a complex physics Rigidbody.
2. **Angle Locking:** Notice how the `CubeController` explicitly zeros out the `pitch` and `roll` components of the Euler angles. This is a common pattern for standard FPS or third-person controllers to prevent the character model from leaning backwards when looking up.

## Exploring the Sample

When you open the `sweeper` project in the editor, look at:

1. **The Scene File (`Assets/scenes/sweeper.scene`):** See how the `CubeController` is attached to a GameObject.
2. **The Inspector:** Notice how the `[Property]` attribute exposes `MoveSpeed` to the editor UI, allowing you to tweak the speed live while the game is running without recompiling.

## Troubleshooting

:::warning Common Gotchas
- **Object moves way too fast or slow:** Ensure you didn't accidentally delete `* Time.Delta` from the equation. If you omit it, your object will move `MoveSpeed` units *per frame*, launching it instantly into the void.
- **Object moves globally instead of locally:** If your object always moves "North" instead of "Forward relative to where it's facing", ensure your movement addition looks like `WorldPosition += WorldRotation * moveDirection` and not `WorldPosition += moveDirection`.
:::

## Related Pages
- [Player Movement](../../how-to/player-movement.md)
- [Detect Input](../../how-to/detect-input.md)
