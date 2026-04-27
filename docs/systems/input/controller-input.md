---
title: "Controller Input"
icon: "🎮"
sources:
  - engine/Sandbox.Engine/Systems/Input/Controller/Input.Controller.cs
  - engine/Sandbox.Engine/Systems/Input/Input.Haptics.cs
  - engine/Sandbox.Engine/Systems/Input/Controller/InputMotionData.cs
  - engine/Sandbox.Engine/Systems/Input/Controller/InputAnalog.cs
  - engine/Sandbox.Engine/Systems/Input/Input.Actions.cs
created: 2026-04-27
updated: 2026-04-27
---

# Controller Input

Specialized Input methods for handling gamepads, including direct analog values, haptics (rumble), and local multiplayer.

## Quick Working Example

```csharp
public sealed class ControllerRumbleComponent : Component
{
	protected override void OnUpdate()
	{
		// Only rumble if they are actually holding a controller
		if ( Input.UsingController && Input.Pressed( "attack1" ) )
		{
			// Trigger a hard impact rumble effect
			Input.TriggerHaptics( HapticEffect.HardImpact );
		}
	}
}
```

### Getting Raw Analog Data
While `Input.AnalogMove` and `Input.AnalogLook` handle standard dual-stick movement, you can query any analog axis directly using `Input.GetAnalog`.

```csharp
// Useful for racing games where you need exact trigger pressure
float gasPedal = Input.GetAnalog( InputAnalog.RightTrigger );
float steering = Input.GetAnalog( InputAnalog.LeftStickX );
```

### Custom Haptics
You can manually set the intensity of the left/right motors and triggers for precise vibration.

```csharp
// Left motor 50%, Right motor 70%, Left Trigger 20%, Right Trigger 0%, for 1000ms
Input.TriggerHaptics( 0.5f, 0.7f, 0.2f, 0f, 1000 );
```

### Motion Controls (Gyro)
If the controller has a gyroscope (like a PlayStation controller), you can read its motion.

```csharp
InputMotionData motionData = Input.MotionData;

// We check if the gyroscope values are non-zero to see if the device supports motion
if ( motionData.Gyroscope != default ) 
{
	Angles angularVelocity = motionData.Gyroscope;
	Vector3 acceleration = motionData.Accelerometer;
	// Apply data as needed...
}
```

### Local Multiplayer (Splitscreen)
You can isolate input checks to a specific controller index using `Input.PlayerScope`.

```csharp
// Check inputs for Player 2 (Index 1)
using ( Input.PlayerScope( 1 ) )
{
	if ( Input.Pressed( "jump" ) )
	{
		// Make Player 2 jump
	}
}
```

## Configuration

| Property/Method | Description |
|-----------------|-------------|
| `Input.UsingController` | `true` if the last input received was from a gamepad. |
| `Input.ControllerCount` | The number of connected gamepads. |
| `Input.GetAnalog(InputAnalog)` | Gets the direct float value (0.0 to 1.0 or -1.0 to 1.0) of a specific stick or trigger. |
| `Input.TriggerHaptics(...)` | Starts a vibration effect on the current controller. |
| `Input.StopAllHaptics()` | Immediately stops all vibration. |
| `Input.MotionData` | A struct containing `Accelerometer` and `Gyroscope` data if the controller supports motion. |
| `Input.PlayerScope(int)` | Scopes subsequent `Input` calls to a specific controller index. |

## Troubleshooting

:::info UsingController State
`Input.UsingController` changes dynamically. If a player wiggles their mouse, it becomes `false`. If they bump their controller's thumbstick, it becomes `true`. Do not cache this value on start; check it every frame.
:::

## Related Pages
* [Input Overview](index.md)
* [Raw Input](raw-input.md)
