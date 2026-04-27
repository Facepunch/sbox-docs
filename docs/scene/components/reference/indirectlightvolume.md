---
title: "Indirect Light Volume (DDGI)"
icon: "🔲"
sources:
  - engine/Sandbox.Engine/Scene/Components/IndirectLighting/IndirectLightVolume.cs
  - engine/Sandbox.Engine/Scene/Components/IndirectLighting/IndirectLightVolume.Relocation.cs
updated: 2026-04-25
created: 2026-04-27
---

# Indirect Light Volume (DDGI)

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `IndirectLightVolume` component.

`IndirectLightVolume` is a Dynamic Diffuse Global Illumination component. It places a 3D grid of light probes inside a bounding box, bakes irradiance and distance data into volume textures, and feeds them to the renderer so shaders can sample indirect lighting at any point inside the volume.

For a feature-level overview see [Indirect Light Volumes](../../../systems/effects/indirect-light-volumes.md). This page documents the component as you'd interact with it from the inspector or code.

## Quick Working Example

```csharp
// Programmatically create a volume around a room and bake it
var go = scene.CreateObject();
go.WorldPosition = roomCenter;

var volume = go.AddComponent<IndirectLightVolume>();
volume.Bounds = BBox.FromPositionAndSize( Vector3.Zero, new Vector3( 1024, 1024, 384 ) );
volume.ProbeDensity = 8;
await volume.BakeProbes();
```

## Common Patterns

### Bake from the editor

Click the **Bake** button on the component (the `[Button( "Bake", "lightbulb" )]` action) to start an asynchronous probe bake. The `Editor > Scene > Bake Indirect Light Volumes` menu item bakes every volume in the scene.

```csharp
await volume.BakeProbes();
```

### Auto-fit to scene

Use the **Fit to Scene Bounds** button (`ExtendToSceneBounds`) to size the volume around every `Renderer`, `Terrain`, and `MeshComponent` in the scene plus 16 units of padding.

### Tuning leaks vs contrast

- Bigger `NormalBias` reduces light leaking onto surfaces from the wrong side at the cost of slightly fading shadows.
- Higher `Contrast` reduces energy conservation per integration pass — use sparingly for a punchier look.

## Properties

| Property | Default | Description |
|---|---|---|
| `Bounds` | `BBox.FromPositionAndSize( 0, 512 )` | Local-space bounding box of the volume. |
| `ProbeDensity` | `8` | Probes per 1024 world units (1–15). |
| `NormalBias` | `5.0` | Bias along surface normals to prevent self-occlusion (0–50). |
| `Contrast` | `1.0` | Energy-conservation multiplier per probe integration pass (1–2). Higher = punchier. |
| `IrradianceTexture` | `null` | Baked color volume texture. Hidden in inspector. |
| `DistanceTexture` | `null` | Baked distance volume texture. Hidden in inspector. |
| `RelocationTexture` | `null` | Baked probe-offset / active-flag volume texture. Hidden in inspector. |

## Read-only state

| Property | Description |
|---|---|
| `ProbeCounts` | `Vector3Int` count of probes along each axis, derived from `Bounds` and `ProbeDensity`. |

## Methods

- `BakeProbes( CancellationToken ct = default )` — start an async bake. Cancels any in-flight bake first. Saves the resulting textures as `.vtex_c` files in the scene folder.
- `ExtendToSceneBounds()` — resize `Bounds` to encompass all scene geometry plus a 16-unit margin.
- `BakeAll()` (static) — bakes every `IndirectLightVolume` in the editor scene.

:::warning "Bake doesn't run at runtime"
`BakeProbes` writes through `Scene.Editor` and saves to the scene folder. Baking is an editor workflow — at runtime the component only consumes the already-baked volume textures.
:::

:::tip "Don't oversize the volume"
Probe counts are clamped to 40 per axis. A 10240-unit volume at density 15 will hit the cap and effectively reduce density anyway. Use multiple smaller volumes for big maps.
:::

## Related Pages

- [Indirect Light Volumes (system overview)](../../../systems/effects/indirect-light-volumes.md)
- [Envmap Probe](envmapprobe.md)
- [Color Ambient Light](ambientlight.md)
