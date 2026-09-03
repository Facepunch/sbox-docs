---
title: "Material"
icon: "💎"
created: 2024-12-05
updated: 2026-09-03
---

# Material

## What is Material API

The Material API is a structure to define the surface properties of what you're drawing as described by traditional PBR shading models.

It does nothing on it's own but it's a contract on how the material would react to lighting, [Shading Models](/rendering/shaders/shading-model.md) consume what comes from it.

## API Reference

```cpp
class Material
{
    float3 Albedo;
    float  Metalness;
    float  Roughness;
    float3 Emission;
    float3 Normal; // World-space normals
    float  TintMask;
    float  AmbientOcclusion;
    float3 Transmission;
    float Opacity;

    // This stuff is part of what describes a surface too and needed for lighting
    float3 WorldPosition;           // vPositionWs (vPositionWithOffsetWs + g_vHighPrecisionLightingOffsetWs.xyz)           
	float3 WorldPositionWithOffset; // vPositionWithOffsetWs in PixelInput
    float4 ScreenPosition;          // vPositionSs in PixelInput

    // baked lighting and/or anisotropic lighting
    float3 TangentNormal;
    float3 WorldTangentU; // vTangentUWs in PixelInput
    float3 WorldTangentV; // vTangentVWs in PixelInput
    float2 LightmapUV; // if D_BAKED_LIGHTING_FROM_LIGHTMAP is enabled

    float2 TextureCoords; // required for ToolsVis
};
```

## Helper Functions

* To initialize an empty material you can use `Material::Init( PixelInput i )` and fill up the needed fields for the [Shading Model](/rendering/shaders/shading-model.md)
* If you are initializing the material in a shader that does not use built-in pixel input struct, but still utilizes standard shading model, you should use `Material::Init( float3 RelativeWorldPosition, float4 ScreenPosition )`. It will automatically setup a material with world/screen coordinate data which is necessary for the shading model.
* `Material::lerp( Material a, Material b, float Amount )` allows to lerp between two materials, works the same way as HLSL's `lerp()` function. 

## Legacy Behavior

While this method is still working and compiles normally, it is recommended to use `Material::Init( PixelInput i )` instead.

* `Material::From( PixelInput i )` can be used to process the [standard input](/rendering/shaders/reference/default-vertex-and-pixel-shader-inputs.md) and automatically expose all common PBR texture inputs to Material Editor. All these texture inputs will be automatically sampled and inserted into initialized material as well. You must add `#include "common/utils/Material.CommonInputs.hlsl"` in your pixel shader code for this to work correctly. If you need more control over inputs, use different ways to initialize a new material. 

