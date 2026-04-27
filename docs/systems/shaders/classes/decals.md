---
title: "Decals (HLSL)"
icon: "🩹"
created: 2026-04-25
updated: 2026-04-25
sources:
  - game/addons/base/Assets/shaders/common/classes/Decals.hlsl
---

# Decals (HLSL)

`Decals::Apply` is what the standard shading model calls to mix all active decal contributions into a fragment's `Material`. If you're writing a custom shading model and you want decals to appear on your geometry, this is the entry point. (For the *gameplay-side* `Decal` component, see [Decal Component](../../effects/decals/decal-component.md).)

```cpp
#include "common/classes/Decals.hlsl"
```

## API

```cpp
static void Decals::Apply( float3 worldPos, in out Material material );

// Lower-level, called per-decal by Apply:
static bool Decals::Resolve(
    Decal decal,
    float3 worldPosition, float3 ddxWorldPos, float3 ddyWorldPos,
    in out Material material
);
```

Both methods *mutate* the `Material` in place — they blend each decal's albedo / normal / roughness / metalness / AO / emission into the material based on the decal's color-alpha mask.

## How `Apply` works

1. Queries the cluster tile at `material.ScreenPosition` for decals overlapping this fragment.
2. Computes world-position derivatives once (`ddx`/`ddy`), shared across all decals at this fragment.
3. For each decal in the cluster, calls `Resolve` to test whether the fragment is inside the decal's volume and, if so, blend it in.

Per-decal `Resolve` performs:

- Quaternion rotation + scale to transform world position into the decal's local space.
- Bounds check against the decal volume (`-0.5 .. 0.5`).
- UV gradient computation from world-position derivatives (proper mipping without `.Sample`).
- Optional **parallax occlusion mapping** if the decal has a height texture.
- Optional **sheet sampling** with frame blending for animated decals.
- Albedo / normal / RMA / emission lookups via `Bindless::GetTexture2D`.
- Angle-based attenuation so decals fade on glancing angles.

## Custom shading model usage

```cpp
#include "common/classes/Decals.hlsl"

float3 ShadeCustom( in out Material m )
{
    // Apply decals BEFORE you compute lighting — decals modify
    // m.Albedo, m.Normal, m.Roughness, m.Metalness, m.AmbientOcclusion, m.Emission
    Decals::Apply( m.WorldPosition, m );

    // ... now run your lighting on the (decal-modified) material
    float3 lit = MyLighting( m );
    return lit;
}
```

The standard `ShadingModelStandard` already calls `Decals::Apply` at the right point — only invoke it manually if you're not using the standard model.

## Notes

- Decal data comes from `DecalBuffer` and `DecalsExtraDataBuffer`, populated by the engine each frame from active `Decal` components in the scene.
- The cluster query is what makes this efficient — only decals whose volumes overlap the current screen tile are iterated, not the entire scene's decal list.
- Animated decals (sheet sequences) read time from `g_flTime` automatically.

## Related

- [Decal Component](../../effects/decals/decal-component.md) — the gameplay-side surface that produces these decals.
- [DecalDefinition](../../effects/decals/decaldefinition.md) — the asset format the textures come from.
- [Bindless API](bindless-api.md) — what the texture lookups go through.
