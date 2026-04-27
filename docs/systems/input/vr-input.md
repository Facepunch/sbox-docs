---
title: "VR Input"
icon: "🥽"
sources:
  - engine/Sandbox.Engine/Systems/Input/VR/Input.Vr.cs
  - engine/Sandbox.Engine/Systems/Input/VR/VRController.cs
  - engine/Sandbox.Engine/Scene/Components/VR/VrTrackedObject.cs
  - engine/Sandbox.Engine/Scene/Components/VR/VrAnchor.cs
created: 2026-04-27
updated: 2026-04-27
---

# VR Input

The VR Input API allows you to access positional and rotational data from the player's Head Mounted Display (HMD) and hand controllers, providing everything you need to build interactive Virtual Reality games.

## Quick Working Example

```csharp
public sealed class VRHands : Component
{
	protected override void OnUpdate()
	{
		// Only run if VR is active
		if ( !Game.IsRunningInVR )
			return;

		// Get the left controller's transform
		Transform leftHand = Input.VR.LeftHand.Transform;

		// Get the right controller's pointing/aiming transform
		Transform rightAim = Input.VR.RightHand.AimTransform;

		// Do something when the user presses the trigger
		if ( Input.VR.RightHand.Trigger.Value > 0.5f )
		{
			Log.Info( "Right Trigger Pulled!" );
		}
	}
}
```

### Accessing VR Transforms in Code

You can access the current transforms for the Head and Hands at any time using `Input.VR`.

*   **`Input.VR.Head`**: Returns the `Transform` of the Head Mounted Display.
*   **`Input.VR.LeftHand`** & **`Input.VR.RightHand`**: Return a `VRController` object containing rich input state and transforms.

Controllers have two primary transforms:
*   `Transform`: The "Grip" pose. This is centered on the palm/grip of the controller. It's best for visualizing hand placement or holding objects.
*   `AimTransform`: The "Aim" pose. This points forward from the controller, similar to holding a laser pointer. It's best for UI interaction or aiming weapons.

```csharp
// Example: Using the Aim transform for a raycast
Transform aim = Input.VR.RightHand.AimTransform;
var trace = Scene.Trace.Ray( aim.Position, aim.Position + aim.Forward * 1000f ).Run();
```

### Setting the VR Anchor

Before using VR features, your scene must have a GameObject defining the center of the playspace. You can do this by attaching the **VR Anchor** component to an empty GameObject.

The `VRAnchor` component automatically updates the `Input.VR.Anchor` property every frame based on its `GameObject.WorldTransform`. All `Input.VR` transforms (Head, LeftHand, RightHand) are automatically returned in World Space relative to this Anchor.

### Tracking Objects Without Code

If you simply want a GameObject (like a 3D model of a hand or a gun) to follow the player's VR controller, you can use the **VR Tracked Object** component instead of writing C#.

1.  Create a GameObject.
2.  Add the `VRTrackedObject` component.
3.  Set the **Pose Source** to `Head`, `LeftHand`, or `RightHand`.
4.  Set the **Pose Type** to `Grip` or `Aim` (for hands).
5.  Set the **Tracking Type** to update Position, Rotation, or both.

## Troubleshooting

:::warning "My VR Hands are stuck at 0,0,0!"
Ensure you have a GameObject with a **VR Anchor** component in your active Scene. If `Input.VR.Anchor` is not being updated, the incoming local VR coordinates cannot be correctly translated into World Space.
:::

:::danger "Input.VR is returning null or throwing exceptions!"
Ensure your game is actually running in VR before accessing `Input.VR` data heavily. You can check this by verifying that `Game.IsRunningInVR` is true.
:::

## Related Pages
* [Input Overview](index.md)
* [Controller Input](controller-input.md)