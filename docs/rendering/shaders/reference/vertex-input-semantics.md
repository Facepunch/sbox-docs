---
title: "Vertex Input Semantics"
icon: "🖨️"
created: 2026-08-25
updated: 2026-09-17
---

# Vertex Input Semantics

Most of the model data is fed to shaders through vertex input struct. Each field in this struct is bound to a specific HLSL register and also **Semantic**, that can be used by the model/renderer/map compiler to decide which data stream gets packed into a specific register. 

Some of this data comes from the model itself, some of it comes from the model compiler, or runtime values. Here's an example of a semantic being used in standard vertex input struct:

```cpp
// Object-space vertex position, "POSITION" is a HLSL register and PosXyz is a semantic.
float3 vPositionOs : POSITION < Semantic( PosXyz ); >;
```

Semantics are not always necessary, it is a s&box-specific thing and isn't native to HLSL after all. You DO need them if your shader will be used for regular models (compiled with ModelDoc) or map geometry, but they might not be necessary if you're creating a procedural mesh. In these situations you will likely have your own vertex layout built, which should be enough to read vertex input data with just HLSL registers.

Here's a full list of all available semantics:

| Semantic Name | Purpose |
|---------------|---------|
| `PosXyz` | Object-space vertex position |
| `Normal` | Plain mesh normals. Not used by standard vertex input struct |
| `OptionallyCompressedTangentFrame` | Compressed normal + tangent, used by standard vertex input struct |
| `LowPrecisionUv` | UV0 channel of the mesh | 
| `LowPrecisionUv1` | UV1 channel of the mesh |
| `LowPrecisionUv2` | UV2 channel of the mesh |
| `LowPrecisionUv3` | UV3 channel of the mesh |
| `TangentU_SignV` | Tangent basis. This data won't be passed to model if it uses normal+tangent compression (which is enabled by default), will become available only if model is explicitly using uncompressed vertices (can be triggered by "Use Expensive Tangents" render markup in ModelDoc) |
| `Color` | Vertex color |
| `BlendWeight` | Used by skinned meshes |
| `BlendIndices` | Used by skinned meshes |
| `VertexPaintBlendParams` | Used by blendable shader, can be assigned by scene map editor/Hammer, not part of standard vertex input struct |
| `VertexPaintTintColor` | Used by blendable shader, can be assigned by scene map editor/Hammer, not part of standard vertex input struct |
| `PerVertexLighting` | Written by lighting baker when a static prop/surface has per-vertex lighting enabled. Mutually exclusive with LightmapUV. Not present in standard vertex input |
| `Curvature` | Available only if "Calculate Per-Vertex Curvature" is enabled from ModelDoc |
| `PropWorldOrigin` | Not used by any shader | 
| `InstanceTransformUv` | Instance ID of this mesh | 
| `MorphIndex` | Morph data ID, used by skinned meshes |

## More UV channels

Compiled models support up to four UV channels, however if you are using default vertex/pixel input structs by default, you will have access only to first two UV channels, (UV0 and UV1) where second one is gated behind `S_UV2` combo unless you just write it to PixelInput struct manually. 

To add UV2 and UV3 to your shader, you must add custom fields in your VertexInput struct, for example:

```cpp
VS
{
    // Adding fields for UV2/UV3 channels on top of standard vertex input
    #include "common/vertexinput.hlsl"

    float2 vTexCoord3 : TEXCOORD4 < Semantic( LowPrecisionUv2 ); >; 
    float2 vTexCoord4 : TEXCOORD5 < Semantic( LowPrecisionUv3 ); >;
}
```
