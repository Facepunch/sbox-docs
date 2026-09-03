---
title: "Dynamic Reflections"
icon: "🪩"
created: 2024-08-16
updated: 2026-09-03
---

# Dynamic Reflections

Much like SSAO, you can composite dynamic specular reflections on your object, whether they are SSR or eventually Raytraced Reflections

You can use it directly by sampling on your shader:

```cpp
float4 DynamicReflections::Sample( float2 ScreenPosition, float Roughness = 0.0f )
```


On the standard shading model they are composited with the correct BRDF, so it respects the reflection value from Metalness and Roughness set by it

Roughness parameter when sampling will optionally get a specific mip level from the reflection texture from `0.0f` to `1.0f` to composite with variable blurriness based on how rough the overlayed material is

`DynamicReflections::IsEnabled()` tells you whether there's a reflection texture bound at all, so you can skip the work when there isn't.

