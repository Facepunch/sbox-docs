---
title: "VR Input"
sources:
  - engine/Sandbox.Engine/Platform/VR/Input/
created: 2026-04-27
updated: 2026-04-27
---

# VR Input

The VR Input system maps the raw physical inputs from VR controllers (like the Valve Index Knuckles, Meta Quest Touch controllers, or HTC Vive Wands) into standardized actions that your game can interpret.

:::info Higher-Level Inputs
For most gameplay logic, you should use the standard [Input System](../../systems/input/index.md) actions which can be bound to VR controllers in the project settings. This API is for querying raw, device-specific VR input states.
:::

## Quick Start

```csharp
using Sandbox;

public sealed class VRAmmoDisplay : Component
{
    protected override void OnUpdate()
    {
        if ( !VR.IsActive ) return;

        // Check if the primary button on the right controller is pressed
        if ( Input.VR.RightHand.ButtonA.IsPressed )
        {
            ReloadWeapon();
        }

        // Read the analog trigger value (0.0 to 1.0)
        float triggerPull = Input.VR.RightHand.Trigger.Value;
        if ( triggerPull > 0.5f )
        {
            FireWeapon();
        }
    }

    private void ReloadWeapon() { /* ... */ }
    private void FireWeapon() { /* ... */ }
}
```

## Core Features

### Button States
You can check if buttons are currently pressed, just pressed this frame, or just released.

### Analog Axes
Triggers and grips return a float value between `0.0` and `1.0`. Thumbsticks and trackpads return a `Vector2` representing the X/Y axes (usually `-1.0` to `1.0`).

### Haptics (Rumble)
You can trigger haptic feedback (vibration) on the controllers to simulate physical interactions, like firing a gun or touching a surface.

## Common Patterns

### Triggering Haptic Feedback
Haptic feedback is crucial for immersion in VR.

```csharp
public void FireWeapon()
{
    // Fire the gun logic...

    // Trigger a short, sharp vibration on the right controller
    // Parameters: duration (seconds), frequency (hz), amplitude (0-1)
    Input.VR.RightHand.TriggerHapticVibration( 0.1f, 200f, 1.0f );
}
```

## Troubleshooting

:::warning Controller Not Found
If `Input.VR.RightHand` returns default/zero values, the controller may be asleep, turned off, or out of tracking bounds. Always write your logic defensively to handle missing controllers gracefully.
:::

## Related Pages
* [VR Platform Integration](./vr.md)
* [Input System](../../systems/input/index.md)
