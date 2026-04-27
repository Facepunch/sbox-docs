---
title: "Depth of Field"
icon: "🎯"
created: 2024-04-15
updated: 2024-04-15
sources:
  - engine/Sandbox.Engine/Scene/Components/PostProcessing/Effects/DepthOfField.cs
---

# Depth of Field

The `DepthOfField` (DoF) post-processing effect simulates the focus properties of a real camera lens. Objects at a specific distance stay sharp, while objects closer or further away become blurred.

## Quick Working Example

```csharp
using Sandbox;

public sealed class FocusCamera : Component
{
    [Property] public GameObject FocusTarget { get; set; }

    protected override void OnUpdate()
    {
        var camera = Scene.Camera;
        if ( camera == null || FocusTarget == null ) return;
        
        var dof = camera.GameObject.Components.GetOrCreate<DepthOfField>();
        
        // Calculate the distance from the camera to our target
        float distance = camera.WorldPosition.Distance( FocusTarget.WorldPosition );
        
        // Keep the target in focus
        dof.FocalDistance = distance;
        dof.FocusRange = 100f; // Give a small window of sharpness around the target
        dof.BlurSize = 20f;
    }
}
```

### Cinematic Cutscenes
During cutscenes, you often want to guide the viewer's eye to a specific character. You can attach a `DepthOfField` component to your cinematic camera and animate the `FocalDistance` to slowly shift focus from the background to a character in the foreground (a "rack focus").

### Aiming Down Sights (ADS)
In first-person shooters, when the player aims down the sights of a weapon, you can apply a subtle DoF effect to slightly blur the weapon itself while keeping the distant enemies sharp.
To do this, you would enable `FrontBlur` and keep the `FocalDistance` far away.

## Configuration

| Property | Description |
|---|---|
| `BlurSize` | How blurry the out-of-focus areas become. Higher values = more blur. |
| `FocalDistance` | The distance from the camera (in world units) where objects are perfectly in focus. |
| `FocusRange` | The distance around the focal point that remains reasonably sharp before falling off into heavy blur. |
| `FrontBlur` | If true, objects *closer* to the camera than the focal distance will be blurred. |
| `BackBlur` | If true, objects *further* from the camera than the focal distance will be blurred. |

## Console Variables

| ConVar | Description |
|---|---|
| `r_dof_quality` | Global quality setting for Depth of Field. `0` = Off, `1` = Low, `2` = Medium, `3` = High. Lowering this can significantly improve performance. |

## Troubleshooting

:::danger "Everything is blurry!"
1. Your `FocalDistance` is probably set to a distance where there are no objects. Try using a Trace (Raycast) from the center of the camera to find the distance to the geometry you are looking at, and set the `FocalDistance` to that value.
2. Your `FocusRange` might be too small (e.g., `0`), meaning only a microscopic slice of the world is in focus. Increase it.
:::

:::warning "It tanks my framerate!"
Depth of Field is an expensive effect because it requires heavy sampling and blurring passes. If you are experiencing low FPS, try reducing the `BlurSize` or globally lowering the `r_dof_quality` console variable.
:::

## Related Pages
- [Post Processing Overview](../index.md)
- [Camera Component](../../../scene/components/reference/cameracomponent.md)