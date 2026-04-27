---
title: "VR"
icon: "🥽"
sources:
  - engine/Sandbox.Engine/Platform/VR/VRSystem.cs
  - engine/Sandbox.Engine/Scene/Components/VR/VrAnchor.cs
  - engine/Sandbox.Engine/Scene/Components/VR/VrTrackedObject.cs
  - engine/Sandbox.Engine/Scene/Components/VR/VrHand.cs
created: 2026-04-27
updated: 2026-04-27

---

# VR

s&box provides native support for Virtual Reality, allowing you to easily track head and hand movements, read inputs, and set up VR playspaces.

## Enabling VR

- **Editor:** launch with a headset connected and SteamVR (or your runtime) running. A **VR** button appears at the top right of the editor — it's on by default.
- **Game:** if a VR headset is plugged in and active when the game starts, the build launches in VR automatically.

A complete reference scene is available on [sbox-scenestaging](https://github.com/Facepunch/sbox-scenestaging) under `test.vr` — useful as a starting point.

## Quick Working Example

```csharp
public class VRShooterComponent : Component
{
	protected override void OnUpdate()
	{
		// Only run this logic if we are actually in VR
		if ( !Game.IsRunningInVR ) return;

		// Check if the player is pulling the left trigger
		if ( Input.VR.LeftHand.Trigger.Value > 0.5f )
		{
			Log.Info( "Shoot!" );
			
			// Vibrate the controller
			Input.VR.LeftHand.TriggerHaptics( new HapticEffect(), lengthScale: 0.5f );
		}
	}
}
```

### Setting up a VR Player

To create a basic VR player that can look around and see their hands:

1. **Create the Root:** Create an empty GameObject called "VR Player". Add a `VR Anchor` component to it. This object acts as the carpet.
2. **Create the Head:** Inside the Root, create a new GameObject called "Head".
    - Add a `VR Tracked Object` component. Set **Pose Source** to `Head`.
    - Add a `Camera` component. Set the **TargetEye** to `StereoTargetEye.Both`.
3. **Create the Hands:** Inside the Root, create two more GameObjects: "Left Hand" and "Right Hand".
    - Add a `VR Tracked Object` to each. Set **Pose Source** to `LeftHand` and `RightHand`.
    - Add a `Model Renderer` to each to draw a temporary box or hand model.

*Note: All Tracked Objects should have **UseRelativeTransform** enabled so they correctly inherit the VR Anchor's position.*

### Reading VR Inputs

Instead of using the standard action-based Input system, VR controllers have specific analog and digital inputs you can query directly via `Input.VR`.

```csharp
protected override void OnUpdate()
{
	// Left Joystick (AnalogInput2D)
	Vector2 moveInput = Input.VR.LeftHand.Joystick.Value;
	
	// Right Joystick button press (DigitalInput)
	bool isJoyPressed = Input.VR.RightHand.JoystickPress.IsPressed;

	// Left and Right Grip (AnalogInput, 0.0 to 1.0)
	float leftGrip = Input.VR.LeftHand.Grip.Value;
	float rightGrip = Input.VR.RightHand.Grip.Value;

	// Face buttons (DigitalInput)
	bool buttonA = Input.VR.RightHand.ButtonA.IsPressed;
	bool buttonB = Input.VR.RightHand.ButtonB.IsPressed;
}
```

### Aiming vs Gripping

When setting up a `VR Tracked Object` for a hand, you can choose the **Pose Type**.
- **Grip**: The transform is centered inside the palm, roughly where the handle of the controller rests. This is best for tracking the physical controller itself.
- **Aim**: The transform points directly forward, like a laser pointer. This is best for raycasting, aiming weapons, or interacting with UI.

### Haptics

You can trigger haptics (controller vibration) on a specific hand.

```csharp
// Simple vibration on the right controller
Input.VR.RightHand.TriggerHaptics( new HapticEffect(), lengthScale: 1.0f, frequencyScale: 1.0f, amplitudeScale: 0.8f );
```

## Troubleshooting

:::warning Not Launching in VR
If your game is not launching in VR, ensure your VR headset is plugged in, turned on, and that SteamVR (or your VR runtime) is active *before* you launch the editor or game. In the Editor, verify the VR button at the top right is enabled.
:::

:::danger Camera Not Tracking
If your camera is rendering the scene but not moving when you turn your head, ensure you have added the `VR Tracked Object` component to the camera GameObject and set it to track the `Head`.
:::

:::danger Hands are floating far away
Make sure your `VR Tracked Object` components have **Use Relative Transform** checked, so they move relative to your `VR Anchor`, rather than acting as absolute world coordinates.
:::

## Related Pages
- [Input Overview](../input/index.md)
- [VR Anchor](vranchor.md)
- [VR Tracked Object](vrtrackedobject.md)
- [VR Hand](vrhand.md)
- [VR Model Renderer](vrmodelrenderer.md)
