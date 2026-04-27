---
title: "Post Processing"
icon: "🖼️"
created: 2024-05-08
updated: 2026-04-12
sources:
  - engine/Sandbox.Engine/Scene/Components/Camera/CameraComponent.PostProcess.cs
  - engine/Sandbox.Engine/Scene/Components/PostProcessing/PostProcessVolume.cs
---

# Post Processing

The Post Processing system allows you to apply full-screen visual effects (like bloom, depth of field, or color correction) to your game camera.

## Quick Working Example

You can enable post-processing directly on your main camera via code.

```csharp
using Sandbox;

public sealed class EnablePostProcessing : Component
{
    protected override void OnStart()
    {
        var camera = Scene.Camera;
        
        if ( camera != null )
        {
            // Enable post-processing on the camera
            camera.EnablePostProcessing = true;
        }
    }
}
```

## Navigation Hub

* [**Built-in Effects**](effects/index.md) - A list of all available built-in post-processing effects.
* [**PostProcessVolume**](postprocessvolume.md) - Learn how to apply localized effects that only activate when the player enters a specific area.
* [**Creating Post Processes**](creating-postprocesses.md) - How to write your own custom shader-based post-processing effects.

### Global Effects

If you want an effect (like Color Grading) to always be active, simply add the specific effect component (e.g., `ColorGrading`) directly to the same GameObject that holds your `CameraComponent`. As long as the camera has `EnablePostProcessing` set to true, it will render the effect.

### Localized Effects

Often you only want an effect when the player enters a certain area (e.g., green tint in toxic gas, or blurry vision underwater). This is achieved using a `PostProcessVolume`. See the [PostProcessVolume guide](postprocessvolume.md) for details.

## Configuration

These properties on the `CameraComponent` control how post-processing is handled for that specific view.

| Property | Description |
|---|---|
| `EnablePostProcessing` | Toggles whether any post-processing effects are rendered by this camera. Default is `true`. |
| `PostProcessAnchor` | By default, the camera's world position is used to determine if it is inside a `PostProcessVolume`. If you assign a GameObject to this property, the camera will use that object's position instead (useful for top-down or third-person games where the camera is outside the volume but the player is inside). |

## Troubleshooting

:::danger "My post-processing effects aren't showing up!"
1. Ensure your `CameraComponent` has `EnablePostProcessing` checked.
2. If using a `PostProcessVolume`, ensure the camera (or the `PostProcessAnchor`) is physically inside the volume bounds.
:::

## Related Pages
- [Camera Component](../../scene/components/reference/cameracomponent.md)
- [Shaders Overview](../shaders/index.md)
