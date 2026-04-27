---
title: "Classes"
icon: "🍎"
created: 2024-12-08
updated: 2026-04-25
sources:
  - game/addons/base/Assets/shaders/common/classes/
---

# Classes

These are HLSL helper classes built into the s&box engine that make advanced rendering features accessible from custom shaders without rebuilding the underlying machinery (bindless texture access, cluster light queries, depth-pyramid sampling, etc.). They live under `game/addons/base/Assets/shaders/common/classes/` and are included via `#include "common/classes/<Name>.hlsl"`.

## Quick Working Example

The `Depth` class reads the scene's depth buffer for effects like soft particles or shoreline fades.

```cpp
#include "common/classes/Depth.hlsl"

float4 MainPs( PixelInput i ) : SV_Target0
{
    // Fetch raw depth at this screen pixel.
    float sceneDepth  = Depth::Get( i.vPositionSs.xy );

    // Convert to view-space linear depth (world units from the camera).
    float linearDepth = Depth::GetLinear( i.vPositionSs.xy );

    // Soft fade against scene geometry.
    float depthDiff = linearDepth - i.vPositionWs.z;
    float alpha     = saturate( depthDiff / 50.0f );

    return float4( 1.0f, 0.0f, 0.0f, alpha );
}
```

## Navigation Hub

### Lighting
* [**Light**](light.md) — iterate through scene lights (dynamic + static) for a custom shading model.
* [**Ambient Light**](ambient-light.md) — diffuse indirect contribution. Dispatches across DDGI, lightmap probe volume, and env-map probes.
* [**EnvMap**](envmap.md) — specular indirect from box-projected, parallax-corrected env-map probes.
* [**Dynamic Reflections**](dynamic-reflections.md) — runtime reflection sampling (SSR or raytraced, depending on what's bound).

### Geometry & screen-space
* [**Depth**](depth.md) — read and reproject the screen depth buffer; reconstruct world position.
* [**Normals & Roughness**](normals.md) — read world-space normals + roughness from the G-Buffer, with a depth-reconstruction fallback.
* [**G-Buffer**](g-buffer.md) — overview of what writes to the G-Buffer and how it composes.
* [**Screen Space Tracing**](screen-space-tracing.md) — cast rays through the depth pyramid for SSR / contact shadows / custom traces.
* [**SSAO**](ssao.md) — sample the screen-space ambient occlusion buffer.
* [**Motion**](motion.md) — motion vectors and temporal-filter helpers (TAA-style accumulation).
* [**Fog**](fog.md) — apply the scene's global fog (gradient / cubemap / volumetric) to a fragment colour.

### Surface effects
* [**Decals**](decals.md) — mix active decals into a fragment's `Material`. Called by the standard shading model; invoke directly only in a custom one.
* [**Texture Sheets**](texture-sheets.md) — UV math for animated flipbook textures.

### Resources
* [**Bindless API**](bindless-api.md) — access thousands of textures without binding slots, by integer index.
