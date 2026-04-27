---
title: "G-Buffer"
icon: "📝"
created: 2025-07-28
updated: 2025-07-28
---

# G-Buffer

Materials that write to Depth PrePass also write to a G-Buffer, if you are using the standard ShadingModel, you just have to make sure you have a `Depth();` mode enabled

![](./images/g-buffer.png)

It contains minimal information about the object before we do the lighting pass, like it's Normals and Roughness, it can be used in post-processing to do all sorts of effects like accurate Ambient Occlusion and Screen-Space Reflections

You can sample the G-Buffer in any shader with

```cpp
static float3 Normals::Sample( int2 screenPos, uint msaaSampleIndex = 0 );
static float3 Normals::SampleFromDepth( int2 screenPos );
static float  Roughness::Sample( int2 screenPos, uint msaaSampleIndex = 0 );
```

`Normals::Sample` returns the world-space normal at that screen pixel, in `-1..1` range. If `NormalsTextureIndex` is unbound, or the object at that texel didn't write a normal, it falls back to `SampleFromDepth` — reconstructing the normal from the depth buffer using a finite-difference cross product. `SampleFromDepth` is also exposed directly if you want to bypass the G-buffer.

`Roughness::Sample` returns the roughness value packed into the alpha channel of the same `DepthNormals` texture (`0..1`).

The optional `msaaSampleIndex` lets MSAA shaders pick a specific sub-sample.
