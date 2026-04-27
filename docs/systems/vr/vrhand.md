---
title: "VR Hand"
icon: "👋"
sources:
  - engine/Sandbox.Engine/Scene/Components/VR/VrHand.cs
created: 2026-04-27
updated: 2026-04-27
---

# VR Hand

The `VRHand` component reads skeletal finger data from SteamVR and automatically poses a `SkinnedModelRenderer` to match the player's physical hand.

## Quick Working Example

```csharp
// The VRHand component handles the complex skeletal math automatically.
// You simply attach it alongside a SkinnedModelRenderer of a hand.
```

### Creating a tracked Hand Model

1. Create a "Left Hand" GameObject.
2. Attach a `SkinnedModelRenderer` and assign a compatible hand model (e.g., `models/hands/alyx_hand_left.vmdl`).
3. Attach the `VRHand` component.
4. Set the `HandSource` to `Left`.
5. Attach a `VRTrackedObject` component, set to track the `LeftHand` to ensure the hand moves with the controller.

## Configuration

| Property | Description |
|:---|:---|
| `SkinnedModelComponent` | The renderer containing the hand model to pose. This is automatically assigned if on the same GameObject. |
| `HandSource` | Which controller to read skeletal data from (`Left` or `Right`). |
| `MotionRange` | Determines if the fingers should curl all the way into a fist, or if they should stop when they physically hit the controller. |

## Troubleshooting

:::warning Fingers aren't moving
If your hand model moves around the world but the fingers are stiff:
1. Ensure the model has a skeleton and is assigned to a `SkinnedModelRenderer` (not a standard `ModelRenderer`).
2. Verify the bone names in your model match the expected Valve standard naming conventions (e.g., `finger_index_0_L`). If they don't match, the component cannot map the data.
:::

:::danger Hand is offset from controller
The `VRHand` component only animates the fingers. It **does not** move the hand through space. You must also attach a `VRTrackedObject` component to update the position and rotation of the hand based on the controller's location.
:::

## Related Pages
- [VR Tracked Object](vrtrackedobject.md)
- [VR Model Renderer](vrmodelrenderer.md)
- [VR Overview](index.md)
