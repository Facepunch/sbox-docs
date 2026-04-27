---
title: "Bloom"
icon: "📷"
created: 2024-04-15
updated: 2024-04-15
sources:
  - engine/Sandbox.Engine/Scene/Components/PostProcessing/Effects/Bloom.cs
---

# Bloom

The `Bloom` post-processing effect creates a glow around bright areas of your image. This simulates the way real-world camera lenses and the human eye behave when exposed to intense light.

## Quick Working Example

```csharp
using Sandbox;

public sealed class SetupBloom : Component
{
    protected override void OnStart()
    {
        var camera = Scene.Camera;
        if ( camera == null ) return;
        
        // Post processing must be enabled on the camera
        camera.EnablePostProcessing = true;
        
        // Add or get the Bloom component
        var bloom = camera.GameObject.Components.GetOrCreate<Bloom>();
        
        // Configure the glow
        bloom.Strength = 2.0f;
        bloom.Threshold = 1.5f;
        bloom.Tint = Color.Red;
    }
}
```

### Glowing Projectiles or Laser Beams
Bloom is perfect for sci-fi weapons.
1. Create a bullet tracer with an unlit or emissive material.
2. In the material editor, set the emission color value above 1.0 (e.g., 5.0).
3. Ensure the scene camera has a `Bloom` effect attached with a `Threshold` lower than your material's emission.

### Scene Transitions
You can animate the `Strength` of the bloom over time to create a "whiteout" effect when the player steps outside into bright sunlight, simulating eyes adjusting to the light.

```csharp
public class FlashbangEffect : Component
{
    [Property] public Bloom CameraBloom { get; set; }
    
    public void Explode()
    {
        // Instantly crank up the bloom
        CameraBloom.Strength = 10f;
    }
    
    protected override void OnUpdate()
    {
        // Smoothly return to normal
        CameraBloom.Strength = CameraBloom.Strength.LerpTo( 1.0f, Time.Delta * 2f );
    }
}
```

## Configuration

| Property | Description |
|---|---|
| `Strength` | The overall intensity of the bloom effect. Higher values make the glow larger and brighter. Set to 0 to disable. |
| `Threshold` | Pixels must be brighter than this value to start glowing. A value of 1.0 means only pixels brighter than pure white (in HDR space) will glow. |
| `Gamma` | Controls the falloff or curve of the bloom. Higher values make the inner core brighter and the outer edges fade quicker. |
| `Tint` | Multiplies the final bloom glow by a specific color. Useful for stylized effects. |
| `Filter` | The filtering method used when downsampling. `Bilinear` is faster, while `Biquadratic` provides smoother, higher-quality blurs. |

## Console Variables

| ConVar | Description |
|---|---|
| `r_bloom` | Globally enable or disable the bloom effect across the entire engine. `1` = enabled, `0` = disabled. |

## Troubleshooting

:::danger "My whole screen is glowing!"
Your `Threshold` is probably too low. If it's below 1.0, normal lit surfaces (like a sunny floor) will start to bloom. Try increasing the `Threshold` to `1.0` or `1.5` so only actual light sources or emissive materials glow.
:::

:::warning "Nothing is blooming even with Strength at 10!"
1. Check that `Threshold` isn't set ridiculously high.
2. Ensure your materials/lights are actually bright enough. An object with a standard albedo color will max out at 1.0 brightness. You need emissive properties to exceed 1.0.
3. Verify that `EnablePostProcessing` is true on your `CameraComponent`.
:::

## Related Pages
- [Post Processing Overview](../index.md)
- [PostProcessVolume](../postprocessvolume.md)