---
title: "Decal"
icon: "📷"
sources:
  - engine/Sandbox.Engine/Scene/Components/Render/Decal.cs
updated: 2026-04-25
created: 2026-04-27
---

# Decal

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `Decal` component.

The `Decal` component projects textures onto opaque or transparent surfaces in the scene. Decals inherit and modify the PBR properties of whatever they're projected onto, so you can use them for bullet impacts, blood, posters, paint splatters, footprints, or any other surface detail that doesn't need its own geometry.

## Quick Working Example

Spawning a decal on hit:

```csharp
public class BulletImpact : Component
{
	[Property] public List<DecalDefinition> Decals { get; set; }

	public void SpawnAt( Vector3 position, Vector3 normal )
	{
		var go = Scene.CreateObject();
		go.WorldPosition = position;
		go.WorldRotation = Rotation.LookAt( -normal );

		var decal = go.AddComponent<Decal>();
		decal.Decals = Decals;
		decal.Size = 8;
		decal.LifeTime = 30f;
		decal.Transient = true; // auto-removed once max decals exceeded
	}
}
```

## Properties

| Property | Description |
|:---|:---|
| `Decals` | List of `DecalDefinition` resources. One is picked at random per spawn. |
| `LifeTime` | Lifetime in seconds. `0` means the decal lives forever. |
| `Looped` | If true the lifetime curve repeats forever. |
| `Transient` | If true the decal is registered with `DecalGameSystem` and gets removed automatically when the global max-decals count is exceeded. Useful for bullet impacts. |
| `Size` | 2D size in world units (the projection box's width and height). |
| `Scale` | Multiplier applied on top of `Size` (animatable curve). |
| `Rotation` | Rotation in degrees around the projection axis (animatable curve). |
| `Depth` | How far the decal projects into the surface, in world units. Default `8`. |
| `Parallax` | Parallax depth strength when a height texture is provided. |
| `ColorTint` | Animatable gradient that tints the albedo. Use it to fade the decal out over its lifetime. |
| `ColorMix` | `0..1`. How strongly the decal's color texture overrides the surface color. `0` = normal/RMO-only decal masked by the color alpha. |
| `AttenuationAngle` | `0..1`. How quickly the decal fades on glancing angles. `0` = no fade. |
| `SortLayer` | `0..255`. Higher layers render over lower ones; ties break by `GameObject.Id`. |
| `SheetSequence` | Enables sprite-sheet sequence selection. |
| `SequenceId` | Which sequence index to pull from the texture sheet. |

## Common Patterns

### Cleaning up after the lifetime ends

The decal scene object stops rendering when its lifetime elapses, but the `GameObject` and `Decal` component remain. For one-shot effects, pair the `Decal` with a `TemporaryEffect` component on the same `GameObject` so the whole hierarchy is cleaned up cleanly.

### Reading world bounds

`WorldBounds` returns a `BBox` covering the projection volume, including any size and scale. This is handy for spatial queries or culling logic.

```csharp
var bbox = decal.WorldBounds;
```

:::warning "Decals don't appear"
Check `Depth` — a decal with `Depth = 0` will never hit anything. Also confirm `Decals` is non-empty; a decal with no `DecalDefinition` will silently disable rendering.
:::

:::note "DecalRenderer is obsolete"
The older `DecalRenderer` is `[Hide]`-marked and obsolete. New code should use `Decal` with `DecalDefinition` resources instead of materials.
:::

## Related Pages

- [Particle Renderers](particle-renderers.md)
- [Sprite Renderer](spriterenderer.md)
