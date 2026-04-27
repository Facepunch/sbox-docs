---
title: "Volumetric Fog Volume"
icon: "👁️"
sources:
  - engine/Sandbox.Engine/Scene/Components/Render/VolumetricFogVolume.cs
created: 2026-04-27
updated: 2026-04-27
---

# Volumetric Fog Volume

The Volumetric Fog Volume component adds a localized, physically-based volumetric fog within a specific bounding box. It interacts with scene lighting to cast volumetric shadows (often referred to as "god rays").

## Quick Working Example

```csharp
public class FogController : Component
{
	[Property] public VolumetricFogVolume Fog { get; set; }

	protected override void OnUpdate()
	{
		if ( !Fog.IsValid() ) return;

		// Pulse the fog strength over time
		Fog.Strength = 0.5f + MathF.Sin( Time.Now ) * 0.5f;
	}
}
```

## Configuration

| Property | Description |
|---|---|
| `Bounds` | The 3D bounding box that contains the volumetric fog. |
| `Strength` | The density or opacity of the fog (0 to 1). Higher values mean thicker fog. |
| `FalloffExponent` | Controls how the fog density fades. A value of 1 provides linear falloff. |

## Troubleshooting

:::danger Performance: High Bounds & Density
Volumetric fog is a demanding rendering effect. Drawing multiple huge bounding boxes with high strength across an entire open world will negatively impact performance. Limit the `Bounds` tightly to the areas where the fog is visible or necessary.
:::

:::warning Fog Interaction
If your fog isn't lighting up, ensure that the lights (e.g., `DirectionalLight`) pointing through the volume have a `FogMode` setting that interacts with volumetrics, and that the fog `Strength` is greater than 0.
:::

## Related Pages
- [Indirect Light Volumes](indirect-light-volumes.md)
