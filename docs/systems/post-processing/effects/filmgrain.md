---
title: "FilmGrain"
icon: "📽️"
created: 2025-10-02
updated: 2025-10-02
sources:
  - engine/Sandbox.Engine/Scene/Components/PostProcessing/Effects/FilmGrain.cs
---

# FilmGrain

The **FilmGrain** component adds simulated film-style grain to the camera output. It's purely visual – it does not affect gameplay or lighting – and is intended to add texture, grit, or stylistic noise to the final image.

![](./images/filmgrain.png)

## Quick Working Example

You can enable FilmGrain dynamically through code on your camera.

```csharp
using Sandbox;

public sealed class VintageCamera : Component
{
	protected override void OnStart()
	{
		var camera = Scene.Camera;
		if ( camera != null )
		{
			// Ensure post processing is enabled
			camera.EnablePostProcessing = true;

			// Add the FilmGrain effect
			var filmGrain = camera.GameObject.Components.GetOrCreate<FilmGrain>();
			filmGrain.Intensity = 0.5f;
			filmGrain.Response = 0.8f;
		}
	}
}
```

## Configuration

| Property | Description |
|----------|-------------|
| **Intensity** | Overall visibility of the grain. 0 = off, 1 = heavy. |
| **Response** | How much the grain reacts to brightness. Higher values = less grain in brighter areas. |

## Troubleshooting

:::danger "The grain is not showing up!"
Ensure that the `CameraComponent` rendering your scene has **Enable Post Processing** checked. Also, verify that `Intensity` is set above `0`.
:::

## Related Pages
- [Post Processing Overview](../index.md)
- [Built-in Effects](index.md)
