---
title: "Gradient Fog"
icon: "🌫️"
sources:
  - engine/Sandbox.Engine/Scene/Components/Render/GradientFog.cs
updated: 2026-04-25
created: 2026-04-27
---

# Gradient Fog

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `GradientFog` component.

`GradientFog` adds a screen-space distance and height fog to the scene. It's a lightweight alternative to volumetric fog: a single component drives one global gradient fog effect over distance from the camera and altitude in the world.

## Quick Working Example

```csharp
var fogGO = Scene.CreateObject();
fogGO.WorldPosition = new Vector3( 0, 0, 0 );

var fog = fogGO.AddComponent<GradientFog>();
fog.Color = new Color( 0.5f, 0.6f, 0.7f, 0.9f );
fog.Height = 256f;
fog.StartDistance = 100f;
fog.EndDistance = 4000f;
```

## Properties

| Property | Description |
|:---|:---|
| `Color` | Fog color. The alpha channel becomes the maximum opacity (`MaximumOpacity`). |
| `Height` | World-units height above this `GameObject` over which the vertical fog fades out. Minimum `1`. |
| `VerticalFalloffExponent` | Shapes the vertical falloff curve. `1` = linear. |
| `StartDistance` | Distance from the camera at which fog starts. |
| `EndDistance` | Distance from the camera at which fog reaches full strength. Default `1024`. |
| `FalloffExponent` | Shapes the distance falloff curve. `1` = linear. |

## Common Patterns

### Layered horizon

Use a low alpha (e.g. `0.4`) on `Color` and a far `EndDistance` for an atmospheric horizon haze that doesn't completely obscure distant geometry.

### Animating fog

Because settings are applied every `OnPreRender`, mutating the properties from another component just works:

```csharp
fog.Color = fog.Color.WithAlpha( MathF.Sin( Time.Now ) * 0.5f + 0.5f );
```

:::warning "Only one GradientFog wins"
There's only one global gradient fog state per scene. If you have multiple enabled `GradientFog` components, the last to render overwrites the others.
:::

## Related Pages

- [Cubemap Fog](cubemapfog.md)
- [Camera Component](cameracomponent.md)
