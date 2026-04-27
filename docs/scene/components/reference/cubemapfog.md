---
title: "Cubemap Fog"
icon: "🌫️"
sources:
  - engine/Sandbox.Engine/Scene/Components/Camera/CubemapFog.cs
updated: 2026-04-25
created: 2026-04-27
---

# Cubemap Fog

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `CubemapFog` component.

The `CubemapFog` component applies a stylized, image-based fog effect to your camera, drawing its colors and lighting directly from a skybox material.

## Quick Working Example

```csharp
using Sandbox;

public sealed class SpookyFogController : Component
{
	[RequireComponent] public CubemapFog Fog { get; set; }

	protected override void OnUpdate()
	{
		// Make the fog pulse between red and blue
		float sine = MathF.Sin( Time.Now * 2f ) * 0.5f + 0.5f;
		Fog.Tint = Color.Lerp( Color.Red, Color.Blue, sine );
		
		// Bring the fog closer to the player
		Fog.StartDistance = 50f;
		Fog.EndDistance = 500f;
	}
}
```

## Usage Patterns

### Automatic Skybox Detection

By default, you do not need to manually set the `Sky` property on the `CubemapFog` component. If you leave it empty (null), the component will automatically find any active `SkyBox2D` component in your scene and use its texture and tint for the fog. This ensures your fog always matches your lighting environment seamlessly.

### Creating Ground Hugging Fog

You can create fog that settles in valleys or along the floor (often called height fog) using the `Height` properties.

```csharp
var fog = Components.GetOrCreate<CubemapFog>();

// Fog starts settling below Z = 500
fog.HeightStart = 500f;

// The transition width of the height fog
fog.HeightWidth = 100f;

// Make the height fog falloff more dramatic
fog.HeightExponent = 2.0f;
```

## Configuration

| Property | Description |
|:---|:---|
| `Sky` | The specific Material containing a skybox texture to use. If left empty, it automatically uses the scene's active `SkyBox2D` component. |
| `Blur` | (0.0 to 1.0) How blurry the sampled cubemap should be. Higher values create smoother, softer fog lighting. Default is `0.5`. |
| `StartDistance` | How far away from the camera the fog should begin appearing. |
| `EndDistance` | The distance at which objects become completely obscured by the fog. |
| `FalloffExponent` | Controls the curve of the distance fog. A value of `1.0` is linear. Higher values make the fog quickly become dense further away. |
| `HeightStart` | The world Z-height at which the fog begins to settle downwards. |
| `HeightWidth` | The thickness/width of the vertical transition area for the height fog. |
| `HeightExponent` | Controls how aggressively the fog thickens below the `HeightStart`. |
| `Tint` | A color multiplier applied to the final fog output. |

## Troubleshooting

:::danger "The component is added but I see no fog!"
The `CubemapFog` component strictly requires a `CameraComponent` to be attached to the same GameObject in order to function. Furthermore, ensure your camera's far clipping plane isn't much shorter than your `EndDistance`.
:::

:::warning "My fog looks pixelated or has sharp details in it."
Increase the `Blur` property. `CubemapFog` works best when the background image is heavily blurred, so the colors blend softly over geometry rather than projecting a sharp image.
:::

## Related Pages
- [Camera Component](cameracomponent.md)
- [Post Processing](../../../systems/post-processing/index.md)
