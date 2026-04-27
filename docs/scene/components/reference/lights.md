---
title: "Lights"
icon: "💡"
sources:
  - engine/Sandbox.Engine/Scene/Components/Light/Light.cs
  - engine/Sandbox.Engine/Scene/Components/Light/DirectionalLight.cs
  - engine/Sandbox.Engine/Scene/Components/Light/PointLight.cs
  - engine/Sandbox.Engine/Scene/Components/Light/SpotLight.cs
  - engine/Sandbox.Engine/Scene/Components/Light/AmbientLight.cs
updated: 2026-04-25
created: 2026-04-27
---

# Lights

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the light components.

Light components illuminate your scene, creating atmosphere, shadows, and visibility for your 3D models.

## Quick Working Example

```csharp
public class StrobeLight : Component
{
	[Property] public PointLight MyLight { get; set; }
	[Property] public float Speed { get; set; } = 5f;

	protected override void OnUpdate()
	{
		if ( MyLight.IsValid() )
		{
			// Make the light color pulse over time
			var intensity = (MathF.Sin( Time.Now * Speed ) + 1f) * 0.5f;
			MyLight.LightColor = Color.Red * intensity;
		}
	}
}
```

## Types of Lights

### Directional Light
A directional light illuminates the entire scene from a specific angle. Its position does not matter, only its rotation. It is primarily used to represent the sun or moon and casts dynamic shadows.

### Point Light
A point light emits light uniformly in all directions from its exact `WorldPosition`. The light fades out based on its `Radius` and `Attenuation`. It is great for lamps, fires, or explosions.

### Spot Light
A spot light emits light in a specific direction within a cone shape. It has both an inner and outer cone angle, allowing the edges of the light to be sharp or soft. Useful for flashlights or stage spotlights.

### Ambient Light
An ambient light adds a flat color to the entire scene, preventing unlit areas from being completely pitch black.

## Configuration

| Property | Applies To | Description |
|---|---|---|
| `LightColor` | Directional, Point, Spot | The primary color and intensity of the light. |
| `FogMode` | Directional, Point, Spot | How this light interacts with volumetric fog. |
| `Radius` | Point, Spot | The distance the light reaches before fading to zero. |
| `Attenuation` | Point | How quickly the light fades out as it approaches the radius (e.g., quadratic falloff). |
| `ConeInner` / `Outer` | Spot | The angles (in degrees) that define the sharp center and soft edge of the spotlight cone. |
| `ShadowCascadeCount` | Directional | Number of cascades to split the view frustum into for dynamic shadows. More cascades = better resolution but higher cost. |
| `ShadowCascadeSplitRatio` | Directional | Controls how cascades are distributed between the first cascade boundary and the far clip. |
| `SkyColor` | Directional | (Legacy) Controls ambient sky color. Use `AmbientLight` instead. |
| `Color` | Ambient | The ambient light color applied everywhere. |

## Troubleshooting

:::danger Performance: Too Many Shadows
Lights that cast dynamic shadows are expensive to render. If you place dozens of `PointLight` or `SpotLight` components with shadows enabled in a single room, your frame rate will drop significantly. Try to bake lighting where possible, or disable shadows on smaller lights.
:::

:::warning Directional Light Position
A `DirectionalLight`'s position does not affect the lighting. Moving it closer to an object won't make the object brighter. Only the GameObject's **Rotation** matters!
:::

## Related Pages
- [Components Overview](../index.md)
