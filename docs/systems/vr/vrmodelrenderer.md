---
title: "VR Model Renderer"
icon: "🥽"
sources:
  - engine/Sandbox.Engine/Scene/Components/VR/VrModelRenderer.cs
created: 2026-04-27
updated: 2026-04-27
---

# VR Model Renderer

The `VRModelRenderer` component automatically fetches and displays the correct 3D model for the specific VR controller the player is holding.

## Quick Working Example

```csharp
// This component works automatically on startup.
// Attach it alongside a ModelRenderer to visualize the physical controller.
```

### Visualizing the Controller

If you want the player to see their actual controllers instead of virtual hands:

1. Create a "Left Controller" GameObject.
2. Attach a `ModelRenderer`.
3. Attach the `VRModelRenderer` component.
4. Set the `ModelSource` to `LeftHand`.
5. Attach a `VRTrackedObject` component to make the GameObject follow the controller's movements.

## Configuration

| Property | Description |
|:---|:---|
| `ModelSource` | Which device to fetch the model for (`LeftHand` or `RightHand`). |
| `ModelRenderer` | The renderer component that will display the fetched model. |

## Troubleshooting

:::warning Showing a Box instead of a Controller
If the component displays a default box (`models/dev/box.vmdl`) instead of a controller, it means SteamVR failed to provide a valid model for the currently connected hardware. Ensure your VR runtime is active and the controllers are powered on and tracking before starting the scene.
:::

## Related Pages
- [VR Hand](vrhand.md)
- [VR Tracked Object](vrtrackedobject.md)
- [VR Overview](index.md)
