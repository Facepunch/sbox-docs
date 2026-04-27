---
title: "2D Skybox"
icon: "👁️"
sources:
  - engine/Sandbox.Engine/Scene/Components/Render/SkyBox2D.cs
updated: 2026-04-25
created: 2026-04-27
---

# 2D Skybox

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `SkyBox2D` component.

The `SkyBox2D` component adds a 2D skybox to the world, rendering a background sky material and optionally providing global indirect lighting.

## Quick Working Example

```csharp
using Sandbox;

public sealed class DayNightCycle : Component
{
    [RequireComponent] public SkyBox2D Sky { get; set; }

    protected override void OnUpdate()
    {
        // Darken the sky over time to simulate night
        float time = MathF.Sin( Time.Now * 0.1f ) * 0.5f + 0.5f;
        Sky.Tint = Color.Lerp( Color.White, Color.Black.WithAlpha( 0.1f ), time );
    }
}
```

### Changing the Sky Material

You can change the sky material at runtime to transition between different times of day or weather conditions.

```csharp
var sky = Components.GetOrCreate<SkyBox2D>();
sky.SkyMaterial = Material.Load( "materials/skybox/skybox_night_01.vmat" );
```

## Configuration

| Property | Description |
|:---|:---|
| `SkyMaterial` | The material used to render the sky. Must contain "sky" in the shader name. |
| `Tint` | A color multiplier applied to the skybox texture and its lighting. |
| `SkyIndirectLighting` | Whether to use this skybox to generate an environment probe for global indirect lighting. Default is true. |

## Troubleshooting

:::warning "Material not updating!"
When assigning a new `SkyMaterial` via code, the engine enforces that the material's shader name must contain the word `"sky"`. If you assign a standard model material, the assignment will be ignored.
:::

## Related Pages
- [Cubemap Fog](cubemapfog.md)
