---
title: "Color Ambient Light"
icon: "👁️"
sources:
  - engine/Sandbox.Engine/Scene/Components/Light/AmbientLight.cs
  - engine/Sandbox.Engine/Scene/Components/Light/Light.cs
updated: 2026-04-25
created: 2026-04-27
---

# Color Ambient Light

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `AmbientLight` component.

`AmbientLight` adds a flat, scene-wide ambient color that fills areas not covered by any light probe. It is the cheapest way to make sure shadowed surfaces are not pitch black when you don't have an `Indirect Light Volume` or baked GI.

`AmbientLight` runs in editor (`Component.ExecuteInEditor`) so the contribution is visible while you are dressing the scene.

## Quick Working Example

```csharp
var ambient = scene.CreateObject().AddComponent<AmbientLight>();
ambient.Color = new Color( 0.3f, 0.32f, 0.4f );
```

## Common Patterns

### Cheap scene fill

For prototype scenes that don't have any GI baked yet, drop in an `AmbientLight` with a soft blue-grey to keep shadows from being unreadable.

```csharp
[Property] public AmbientLight Ambient { get; set; }

protected override void OnStart()
{
    Ambient.Color = Color.Lerp( Color.Black, Color.White, 0.15f );
}
```

### Tinting indoor areas

Combine an `AmbientLight` (or an authored override on a region) with `EnvmapProbe`s for indoor lighting that reads correctly even where direct lights don't reach.

## Properties

| Property | Default | Description |
|---|---|---|
| `Color` | `Color.Gray` | Ambient color applied globally outside of all light probes. |

:::warning "Multiple AmbientLights"
The component does not enforce singletons. If you enable multiple `AmbientLight`s in a scene, the result depends on the rendering pipeline's accumulation — prefer just one.
:::

:::tip "Use light probes for variation"
`AmbientLight` is uniform across the entire scene. If you need ambient color to differ between rooms, place `IndirectLightVolume` (DDGI) or baked `EnvmapProbe`s — the ambient light only fills space outside those probes.
:::

## Related Pages

- [Lights](lights.md)
- [Envmap Probe](envmapprobe.md)
- [Indirect Light Volume](indirectlightvolume.md)
