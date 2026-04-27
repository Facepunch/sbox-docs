---
title: "VR Anchor"
icon: "⚓"
sources:
  - engine/Sandbox.Engine/Scene/Components/VR/VrAnchor.cs
updated: 2026-04-25
created: 2026-04-27
---

# VR Anchor

The `VRAnchor` component updates the VR playspace center based on a GameObject's transform, defining the physical bounds in the virtual world.

## Quick Working Example

```csharp
public class VRTether : Component
{
    protected override void OnStart()
    {
        if ( !Game.IsRunningInVR ) return;
        
        // This implicitly assumes the GameObject has a VRAnchor attached
        Log.Info( "VR Anchor established at: " + GameObject.WorldPosition );
    }
}
```

### Setting up a VR Playspace

Typically, you attach a `VRAnchor` to an empty "VR Player" root object. 

1. Create a "VR Player" root GameObject.
2. Add the `VRAnchor` component to it.
3. Add child objects for the Head and Hands, using the `VRTrackedObject` component, ensuring they are set to use relative transforms.

When you want to teleport or move the player via a joystick, you simply change the `WorldPosition` of the "VR Player" root object.

## Configuration

The `VRAnchor` component does not have any customizable properties in the Inspector. It simply reads the `Transform` of the GameObject it is attached to and updates the global `Input.VR.Anchor` value every frame.

## Troubleshooting

:::danger Tracked Objects are offset incorrectly
If your Head or Hands appear in the wrong location relative to where you expect them to be, ensure that the GameObjects representing them have their `VRTrackedObject` component configured with **Use Relative Transform** checked. Without this, they will track relative to world origin (0,0,0) rather than your `VRAnchor`.
:::

:::warning Only use one VRAnchor
You should only have one active `VRAnchor` component in your scene at a time. Having multiple anchors will cause them to fight over setting the global `Input.VR.Anchor` position every frame, leading to extreme camera jitter.
:::

## Related Pages
- [VR Tracked Object](vrtrackedobject.md)
- [VR Overview](index.md)
