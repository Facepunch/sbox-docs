---
title: "Dynamic Reflections"
icon: "🪩"
created: 2024-08-16
updated: 2025-07-30
---

# Dynamic Reflections

Much like SSAO, you can composite dynamic specular reflections on your object, whether they are SSR or eventually Raytraced Reflections

You can use it directly by sampling on your shader:

```cpp
float3 DynamicReflections::Sample( float4 ScreenPosition, float Roughness )
```


On the standard shading model they are composited with the correct BRDF, so it respects the reflection value from Metalness and Roughness set by it


## Custom Dynamic Reflections

You can implement your own dynamic reflections solution simply by providing the dynamic reflection texture globally from your own component:

```csharp
commands.SetGlobal( "ReflectionColorIndex", PlanarReflection.ColorIndex );
```


Roughness parameter when sampling will optionally get a specific mip level from your texture from `0.0f` to `1.0f` to composite with variable blurriness based on how rough the overlayed material is

## Overriding Indirect Specular Reflections For Disabling

Since you are overriding the envmap reflections of an object, overriding reflection color can also be used to force them off entirely

```csharp
commands.SetGlobal( "ReflectionColorIndex", Texture.Black.ColorIndex );
```
