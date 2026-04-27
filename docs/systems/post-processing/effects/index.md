---
title: "Built-in Effects"
icon: "📁"
created: 2025-10-02
updated: 2025-10-02
sources:
  - engine/Sandbox.Engine/Scene/Components/PostProcessing/Effects/Bloom.cs
  - engine/Sandbox.Engine/Scene/Components/PostProcessing/Effects/MotionBlur.cs
---

# Built-in Effects

s&box provides a wide variety of built-in post-processing effects out of the box. These components can be added to your Camera or a `PostProcessVolume` to enhance the visual quality of your game without needing to write custom shaders.

## Quick Working Example

You can enable and configure built-in effects via code by adding them to your camera's GameObject.

```csharp
using Sandbox;

public sealed class EnableEffects : Component
{
	protected override void OnStart()
	{
		var camera = Scene.Camera;
		if ( camera == null ) return;
		
		// Ensure post-processing is enabled on the camera
		camera.EnablePostProcessing = true;
		
		// Add Bloom effect
		var bloom = camera.GameObject.Components.GetOrCreate<Bloom>();
		bloom.Strength = 1.5f;
		
		// Add Motion Blur effect
		var motionBlur = camera.GameObject.Components.GetOrCreate<MotionBlur>();
		motionBlur.Scale = 0.5f;
	}
}
```

## Available Effects

| Effect | Description |
|---|---|
| `AmbientOcclusion` | Adds an approximation of ambient occlusion using Screen Space Ambient Occlusion (SSAO). It darkens areas where ambient light is generally occluded from such as corners, crevices and surfaces that are close to each other. |
| `BlitOverlay` | Draw a material over the screen. |
| [`Bloom`](bloom.md) | Applies a bloom effect to the camera. |
| `Blur` | Applies a blur effect to the camera. |
| `ChromaticAberration` | Applies a chromatic aberration effect to the camera. |
| `ColorAdjustments` | Applies color adjustments to the camera. |
| `ColorGrading` | Applies color grading to the camera. |
| [`DepthOfField`](depthoffield.md) | Applies a depth of field effect to the camera. |
| [`FilmGrain`](filmgrain.md) | Applies a film grain effect to the camera. |
| `Highlight` | This should be added to a camera that you want to outline stuff. |
| `HighlightOutline` | This component should be added to stuff you want to be outlined. You will also need to add the Highlight component to the camera you want to render the outlines. |
| `MotionBlur` | Applies a motion blur effect to the camera. |
| `Pixelate` | Applies a pixelate effect to the camera. |
| `ScreenSpaceReflections` | Screen-Space Reflections. |
| `Sharpen` | Applies a sharpen effect to the camera. |
| [`Tonemapping`](tonemapping.md) | Remaps HDR colors to LDR for display. Multiple operators available (e.g. ACES, Reinhard). |
| `Vignette` | Applies a vignette to the camera. |

## Troubleshooting

:::danger "Effects aren't showing up!"
Ensure that the `CameraComponent` rendering your scene has `EnablePostProcessing` set to true.
:::

## Related Pages
- [Post Processing Overview](../index.md)
- [PostProcessVolume](../postprocessvolume.md)
