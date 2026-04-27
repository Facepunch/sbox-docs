---
title: "VR Tracked Object"
icon: "🎬"
sources:
  - engine/Sandbox.Engine/Scene/Components/VR/VrTrackedObject.cs
created: 2026-04-27
updated: 2026-04-27
---

# VR Tracked Object

The `VRTrackedObject` component links a GameObject's Transform to a physical VR device, making it move and rotate as the player moves their head or hands.

## Quick Working Example

```csharp
// Attach this component to a GameObject to make it track a VR device automatically.
```

### Setting up the Camera

To make your game render from the player's perspective:
1. Add a `Camera` component to a GameObject.
2. Add a `VRTrackedObject` component to the same GameObject.
3. Set `PoseSource` to `Head`.
4. Check **Use Relative Transform**.

### Setting up Hands

To make objects follow the player's hands:
1. Create a GameObject.
2. Add a `VRTrackedObject` component.
3. Set `PoseSource` to `LeftHand` or `RightHand`.
4. Check **Use Relative Transform**.

## Configuration

| Property | Description |
|:---|:---|
| `PoseSource` | Which physical device to track (`Head`, `LeftHand`, or `RightHand`). |
| `PoseType` | For hands: `Grip` centers on the palm (best for holding objects). `Aim` points forward like a laser (best for UI or guns). |
| `TrackingType` | Choose whether to track `Position`, `Rotation`, `None`, or `All`. |
| `UseRelativeTransform` | **Crucial:** If true, coordinates are local to the `VRAnchor` (the playspace). If false, they are absolute world coordinates. Usually, this should be **true**. |

## Troubleshooting

:::danger Objects aren't moving with the player
If your camera or hands are floating at world origin `(0,0,0)` and ignoring your physical movements, ensure:
1. The game is actually running in VR mode.
2. Your VR headset is awake and tracking.
:::

:::warning Hands are floating far away from the body
If your `VRAnchor` (playspace center) is moved, but your hands don't move with it, you likely forgot to check **Use Relative Transform**. If unchecked, the hands will always track relative to the world center `(0,0,0)`, ignoring teleportation or joystick movement.
:::

## Related Pages
- [VR Anchor](vranchor.md)
- [VR Overview](index.md)
