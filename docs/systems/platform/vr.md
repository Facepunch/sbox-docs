---
title: "VR Platform Integration"
sources:
  - engine/Sandbox.Engine/Platform/VR/
created: 2026-04-27
updated: 2026-04-27
---

# VR Platform Integration

The VR Platform system provides the low-level connection between the s&box engine and OpenXR runtimes (like SteamVR or Oculus). It handles headset tracking, controller inputs, and rendering to the VR display.

:::info Higher-Level Components Available
While this page covers the low-level platform APIs, most game developers should use the higher-level [VR Components](../../systems/vr/index.md) (such as `VRAnchor` or `VRHand`) provided in the Scene system instead.
:::

## Quick Start

```csharp
using Sandbox;

public sealed class VRStatusComponent : Component
{
    protected override void OnUpdate()
    {
        // Check if VR is active and rendering
        if ( VR.IsActive )
        {
            var headPos = VR.Head.Position;
            var headRot = VR.Head.Rotation;
            
            // Log raw head tracking position
            Log.Info( $"Head at: {headPos}" );
        }
    }
}
```

## Core Subsystems

### Input Tracking
The VR Platform handles retrieving the raw 6DOF (Degrees of Freedom) poses for the user's head, hands, and other tracked devices. It also maps physical controller buttons and analog sticks to the engine's input system.

### Overlays
The platform supports creating 2D UI overlays that render "floating" in the VR world, independent of the 3D scene geometry. This is commonly used for system menus or HUDs.

### Rendering
The platform negotiates the required swapchains and eye buffers with the OpenXR runtime to ensure the game renders correctly for stereoscopic vision.

## Common Patterns

### Checking VR State
You should always check if VR is actually active before running VR-specific logic, as the user might launch the game in desktop mode.

```csharp
public sealed class VRPlayerController : Component
{
    protected override void OnUpdate()
    {
        if ( !VR.IsActive )
        {
            // Fallback to standard desktop movement
            DesktopMovement();
            return;
        }

        // Handle VR movement (teleport, smooth locomotion, etc.)
        VRMovement();
    }
}
```

## Troubleshooting

:::warning OpenXR Runtime
If `VR.IsActive` remains false despite a headset being connected, the user's OpenXR runtime may not be configured correctly. For SteamVR users, they need to ensure SteamVR is set as the active OpenXR runtime in the SteamVR settings.
:::

## Related Pages
* [VR Components](../../systems/vr/index.md)
* [Platform Integration](./index.md)
