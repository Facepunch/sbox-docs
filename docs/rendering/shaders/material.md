---
title: "Material"
icon: "💎"
created: 2024-12-05
updated: 2024-12-16
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
    float3 Emission; // Emissive color
    float3 Normal; // World normal
    float  TintMask;
    float  AmbientOcclusion;
    float3 Transmission; // This should probably be TransmissiveMask
    float Opacity;

    //
    // This stuff is part of what describes a surface too and needed for lighting
    //
    float3 WorldPosition;
	float3 WorldPositionWithOffset;
    float4 ScreenPosition; // SV_Position
    float3 GeometricNormal;

    // baked lighting and/or anisotropic lighting
    float3 TangentNormal;
    float3 WorldTangentU;
    float3 WorldTangentV;    
    float2 LightmapUV; // if D_BAKED_LIGHTING_FROM_LIGHTMAP

    float2 TextureCoords; // if TOOL_VIS
};
```

## Helper Functions

* `Material::From( PixelInput i )` can be used to process the [standard input](/rendering/shaders/reference/default-vertex-and-pixel-shader-inputs.md) and expose textures to be used directly to the Material Editor ( MET )
* To initialize an empty material you can use `Material::Init()` and fill up the needed fields for the [Shading Model](/rendering/shaders/shading-model.md)
