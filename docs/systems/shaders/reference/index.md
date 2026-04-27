---
title: "Reference"
icon: "📚"
created: 2023-11-26
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Scene/Components/Render/ModelRenderer.cs
---

# Reference

The Shader Reference section provides comprehensive technical documentation for the HLSL built-in variables, input structures, and helper functions available when writing custom shaders in s&box.

## Quick Working Example

```cpp
#include "common/math.hlsl"

float4 MainPs( PixelInput i ) : SV_Target0
{
    // Access global variables and pixel inputs directly
    float timeOffset = g_flTime;
    float3 worldPos = i.vPositionWs;
    
    // Use vertex helpers to create an animated wave effect
    float wave = sin( worldPos.x + timeOffset );
    
    return float4( wave, 0.0, 0.0, 1.0 );
}
```

## Navigation Hub

* [**Default Vertex and Pixel Shader Inputs**](default-vertex-and-pixel-shader-inputs.md) - Details on the structure and contents of the default input structures passed between shader stages (`PixelInput` and `VertexInput`).
* [**Global Variables**](global-variables.md) - Documentation on globally accessible variables such as time (`g_flTime`), camera position, and viewport resolution.
* [**Screen-Space Reflections**](screen-space-reflections.md) - Information on how to sample and utilize screen-space reflections in your custom shaders.
* [**Vertex Helpers**](vertex-helpers.md) - Helper functions and macros to simplify vertex transformations and calculations.

When writing custom shaders, the most common workflow involves taking the `VertexInput` provided by the model, transforming it into a `PixelInput` using the [Vertex Helpers](vertex-helpers.md), and then manipulating the `PixelInput` data (like world position or normals) in the Pixel Shader.

## Troubleshooting

:::danger Missing Include Files
If you try to use a global variable like `g_flTime` or a helper structure like `PixelInput` and the compiler complains they are undefined, ensure you have included the correct base headers at the top of your shader file (e.g., `#include "common/math.hlsl"`).
:::
