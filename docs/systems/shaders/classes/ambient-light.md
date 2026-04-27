---
title: "Ambient Light"
icon: "🌥️"
created: 2026-04-25
updated: 2026-04-25
sources:
  - game/addons/base/Assets/shaders/common/classes/AmbientLight.hlsl
---

# Ambient Light

The `AmbientLight` HLSL class returns the diffuse indirect lighting at a world position. It transparently dispatches across the three indirect-lighting backends s&box supports (DDGI, lightmap probe volume, env-map probe), so a custom shading model can call one function and get the right answer regardless of which is active in the current scene.

```cpp
#include "common/classes/AmbientLight.hlsl"
```

## API

```cpp
enum AmbientLightKind
{
    EnvMapProbe,            // Image-based lighting (default)
    LightMapProbeVolume,    // Baked probe-volume lighting
    DDGI                    // Dynamic Diffuse Global Illumination
};

static AmbientLightKind AmbientLight::GetKind();
static float3 AmbientLight::From( float3 WorldPosition, float4 PositionSs, float3 WorldNormal );

// Direct backend access if you need to bypass dispatch:
static float3 AmbientLight::FromDDGI( float3 WorldPosition, float3 WorldNormal );
static float3 AmbientLight::FromEnvMapProbe( float3 WorldPosition, float4 PositionSs, float3 WorldNormal );
static float3 AmbientLight::FromLightMapProbeVolume( float3 WorldPosition, float3 WorldNormal );
```

`From` is what you call from a custom shading model. It looks at the scene's lighting state and routes to whichever backend is active:

- **DDGI** — when `DDGI::IsEnabled()` is true.
- **LightMap Probe Volume** — when `UsesBakedLightingFromProbe` is set on the material.
- **EnvMap Probe** — fallback. Iterates env-map probes in the cluster tile and box-projects each one's irradiance.

`GetKind()` returns the enum so you can branch in places where the cost difference matters (DDGI is more expensive than reading a baked probe).

## Custom shading model usage

```cpp
#include "common/classes/AmbientLight.hlsl"

float3 ShadeCustom( Material m, float4 vPositionSs )
{
    float3 ambient = AmbientLight::From( m.WorldPosition, vPositionSs, m.Normal );
    float3 directLight = /* iterate Light::From( ... ) */;

    return m.Albedo * ( ambient + directLight );
}
```

`AmbientLight::From` already includes the global `AmbientLightColor` (set per-view) blended in for the EnvMap path. You don't need to add an extra constant on top.

## Notes

- The standard `ShadingModelStandard` already calls this — you only need to invoke it directly when you're writing a fully custom shading model.
- For *specular* indirect lighting (reflections from env-map probes), use [EnvMap](envmap.md) instead. `AmbientLight` is diffuse-only.

## Related

- [Light](light.md) — direct lighting (dynamic + static).
- [EnvMap](envmap.md) — specular indirect / reflection probes.
- [Dynamic Reflections](dynamic-reflections.md) — runtime-computed reflections (SSR/Raytraced).
