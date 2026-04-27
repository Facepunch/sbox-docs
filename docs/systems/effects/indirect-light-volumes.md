---
title: Indirect Light Volume (DDGI)
sources:
  - engine/Sandbox.Engine/Scene/Components/IndirectLighting/IndirectLightVolume.cs
created: 2026-04-27
updated: 2026-04-27
---

# Indirect Light Volume (DDGI)

The `IndirectLightVolume` component provides dynamic diffuse global illumination (DDGI) using a 3D grid of probes to bounce lighting naturally around your scene. 

## Quick Example

While typically placed via the Editor, you can dynamically create and configure an `IndirectLightVolume` in code:

```csharp
using Sandbox;

public class LightingSetup : Component
{
    protected override void OnStart()
    {
        var ddgi = Scene.Components.Create<IndirectLightVolume>();
        
        // Define the area to cover with probes
        ddgi.Bounds = BBox.FromPositionAndSize( Vector3.Zero, new Vector3( 1024.0f ) );
        
        // Adjust the density of probes (higher = better quality, lower performance)
        ddgi.ProbeDensity = 4; 
        
        // Make the probes expand to automatically encompass the scene bounds
        ddgi.ExtendToSceneBounds();
    }
}
```

## How It Works

Think of the **Indirect Light Volume** like a net of invisible sensors (probes) cast over your level. Instead of baking light maps where shadows and bounced light are frozen as a texture, these probes continuously sample the light and geometry around them. As a result, moving lights or changing time-of-day will dynamically bounce off walls and bleed colors onto adjacent surfaces in real-time.

### Adjusting Probe Density
For performance, you want the lowest density that still looks good. You can reduce `ProbeDensity` in wide-open areas and increase it in tight interiors where light bounces are more complex.

### Handling Probes Inside Geometry
When placing a volume, some probes will inevitably fall directly inside a wall or solid object, which can cause ugly lighting artifacts (leaking blackness or intense bright spots). 
The `InsideGeometryBehavior` setting lets you manage this:
- **Relocate:** The engine will attempt to nudge the occluded probe slightly out of the wall so it can sample light properly.
- **Deactivate:** The engine simply turns off the occluded probe. This seals light leaks entirely but might leave small gaps in the lighting data.

## Configuration

| Property | Description |
|---|---|
| `Bounds` | The world-space bounding box that defines the volume's coverage area. |
| `ProbeDensity` | The number of probes per 1024 world units. (Range: 1 to 15). Higher values increase resolution at a performance cost. |
| `NormalBias` | Adjusts how probe sampling reacts to surface normals to prevent self-shadowing artifacts. |
| `Contrast` | Adjusts the contrast of the bounced lighting. |

## Troubleshooting

:::warning Probes inside geometry causing black splats
If you notice small, unlit black patches or completely black lighting on objects near walls, it means probes are stuck inside geometry.

**Fix:** Change the `InsideGeometryBehavior` to `Deactivate` to kill those probes, or use `Relocate` to push them outward. If this fails, consider adjusting the `Bounds` or `ProbeDensity` so probes align better with open space.
:::

:::danger Extremely low frame rates
Placing an `IndirectLightVolume` over a massive open world with a `ProbeDensity` of 10+ will drastically consume GPU memory and frame time.

**Fix:** Lower `ProbeDensity`. If you have a massive map, consider using multiple smaller, lower-density volumes for wide open areas, and higher-density ones restricted only to specific interiors.
:::

## Related Pages
- [Lights](../../scene/components/reference/lights.md)