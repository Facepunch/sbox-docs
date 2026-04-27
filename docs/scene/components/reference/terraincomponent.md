---
title: "Terrain"
icon: "🏞️"
sources:
  - engine/Sandbox.Engine/Scene/Components/Terrain/Terrain.cs
  - engine/Sandbox.Engine/Scene/Components/Terrain/Terrain.Properties.cs
  - engine/Sandbox.Engine/Scene/Components/Terrain/Terrain.Collider.cs
updated: 2026-04-25
created: 2026-04-27
---

# Terrain

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `Terrain` component.

`Terrain` renders heightmap-based terrain and acts as a heightfield collider. It is a `Collider` subclass, so it participates in the physics system the same way any other collider does — but it builds its physics shape from the terrain's heightmap rather than from primitive shapes. The component is `[ExecuteInEditor]` so the terrain renders and collides while you're editing it.

## Quick Working Example

```csharp
public class TerrainSetup : Component
{
    [Property] public TerrainStorage MyHeightmap { get; set; }

    protected override void OnStart()
    {
        var terrain = AddComponent<Terrain>();
        terrain.Storage = MyHeightmap;
        terrain.TerrainSize = 16384f;
        terrain.TerrainHeight = 2048f;
    }
}
```

## Common Patterns

### Raycasting against the heightfield in code

`RayIntersects` does a CPU heightfield trace against the terrain's data, returning a world position. This is useful for editor tools (the gizmo uses it for hover detection) and gameplay code that needs a height query without going through the physics system.

```csharp
if ( terrain.RayIntersects( camera.ScreenPixelToRay( Mouse.Position ), 5000f, out var hit ) )
{
    Log.Info( $"Mouse over terrain at {hit}" );
}
```

### Toggling collision

`EnableCollision` rebuilds the heightfield shape on assignment. This is also how `IndirectLightVolume.ExtendToSceneBounds` temporarily forces collision on so the volume can read the terrain's world bounds.

```csharp
terrain.EnableCollision = false; // disables the heightfield shape
```

### Overriding the terrain shader

Set `MaterialOverride` to a custom material that implements the Terrain Shader API to fully replace the rendering pipeline.

## Properties

| Property | Default | Description |
|---|---|---|
| `Storage` | `null` | The `TerrainStorage` resource (heightmap, control map, materials). Reassigning rebuilds everything. |
| `MaterialOverride` | `null` | Custom material that overrides the default terrain rendering shader. |
| `TerrainSize` | (from `Storage`) | World-space width/length of the terrain. |
| `TerrainHeight` | (from `Storage`) | World-space maximum height. |
| `ClipMapLodLevels` | `6` | Number of clipmap LOD rings. |
| `ClipMapLodExtentTexels` | `256` | Texel extent per clipmap LOD ring. |
| `SubdivisionFactor` | `1` | Subdivision multiplier for the clipmap mesh (1–4). |
| `SubdivisionLodCount` | `3` | How many LODs use the subdivided mesh (1–6). |
| `RenderType` | `ShadowRenderType.Off` | Shadow rendering mode (Off / On / ShadowsOnly). |
| `EnableCollision` | `true` | Whether the heightfield collider is generated. |

`Storage`, `EnableCollision`, and the clipmap properties all rebuild GPU buffers / collision shapes when changed.

## Read-only / inherited

`Terrain` inherits from `Collider`, so the standard collider settings (`Surface`, tags) apply. `IsConcave` is hard-coded `true`.

## Methods

- `Create()` — rebuild GPU resources and collider. Called automatically when `Storage` changes or the component is enabled.
- `RayIntersects( Ray ray, float distance, out Vector3 position )` — CPU heightfield raycast. `position` is in world space.

:::warning "No Storage = invisible"
With `Storage == null` the component does nothing — no draws, no collision. Make sure the terrain resource is assigned.
:::

:::warning "Headless servers skip rendering"
Inside `Create`, the GPU/clipmap path is gated on `!Application.IsHeadless`. Servers still build the collision shape but do not allocate textures or scene objects.
:::

:::tip "Per-pixel material at runtime"
The control map is exposed as a GPU texture; sampling it from a shader gives you the same material id used by the physics shape, so you can match VFX or footstep sounds to terrain material without a separate trace.
:::

## Related Pages

- [Hammer Mesh](hammer-mesh.md)
- [Indirect Light Volume](indirectlightvolume.md)
