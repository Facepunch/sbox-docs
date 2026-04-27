---
title: "EnvMap"
icon: "🪞"
created: 2026-04-25
updated: 2026-04-25
sources:
  - game/addons/base/Assets/shaders/common/classes/EnvMap.hlsl
---

# EnvMap

`EnvMap::From` returns specular indirect lighting from the active environment-map probes — parallax-corrected, roughness-aware, blended across overlapping probes by edge feathering. This is the shader-side counterpart of the `EnvmapProbe` scene component.

```cpp
#include "common/classes/EnvMap.hlsl"
```

## API

```cpp
static float3 EnvMap::From(
    float3 WorldPosition,
    float4 PositionSs,
    float3 WorldNormal,
    float2 Roughness = 0.0f
);
```

Returns the linear-space colour contribution from cubemap reflections at the given world position. The optional `Roughness` parameter selects an appropriate mip from the env-map (sqrt'd internally), so you can blur reflections on rough surfaces without doing your own filtering.

## How it works

1. Queries the cluster tile at `PositionSs` for env-map probes affecting this fragment.
2. For each probe, transforms the world position into local cubemap space and computes a box-projected reflection vector (`CalcParallaxReflectionCubemapLocal`).
3. Samples the cubemap at the appropriate mip level for the requested roughness.
4. Accumulates probe contributions weighted by edge feathering — closer-to-centre probes contribute more, fading at the boundary.
5. Stops accumulating once `flDistAccumulated >= 1.0`.

If no probes affect this fragment, returns `0`.

## Custom shading model usage

```cpp
#include "common/classes/EnvMap.hlsl"
#include "common/classes/AmbientLight.hlsl"

float3 ShadeCustom( Material m, float4 vPositionSs )
{
    float3 diffuseIndirect  = AmbientLight::From( m.WorldPosition, vPositionSs, m.Normal );
    float3 specularIndirect = EnvMap::From( m.WorldPosition, vPositionSs, m.Normal, m.Roughness );

    // ... blend with direct lighting using your BRDF
    return m.Albedo * diffuseIndirect + specularIndirect;
}
```

## Notes

- The `Roughness` parameter is `float2` to support anisotropic roughness — pass the same value in both channels for isotropic surfaces.
- Box projection means env-map probes work correctly inside rooms even if the cubemap was captured from one position. Sample-to-room geometry is corrected automatically.
- For dynamic reflections (SSR / raytraced), use [Dynamic Reflections](dynamic-reflections.md) — `EnvMap` is for the *baked* indirect contribution; the standard shading model composites them.

## Related

- [Ambient Light](ambient-light.md) — diffuse indirect counterpart.
- [Dynamic Reflections](dynamic-reflections.md) — real-time reflection sampling.
- [Envmap Probe component](../../../scene/components/reference/envmapprobe.md) — the scene-side surface that generates these probes.
