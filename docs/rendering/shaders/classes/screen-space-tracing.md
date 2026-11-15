---
title: "Screen Space Tracing"
icon: "🥢"
created: 2024-12-07
updated: 2026-11-15
---

# Screen Space Tracing

`ScreenSpace::Trace` provides functionality for tracing rays in screen space to compute effects like [Dynamic Reflections](/rendering/shaders/classes/dynamic-reflections.md) or any kind of ray in world space. It leverages hierarchical ray marching for efficient performance.

## API Reference

```cpp
static TraceResult ScreenSpace::Trace(
    const float3 Position,  // The world-space origin point of the trace.
    const float3 Direction, // The world-space direction vector of the ray.
    uint nMaxSteps = 64     // Maximum steps for ray marching.
);
```

The TraceResult structure represents the outcome of a trace operation:

| Member | Type | Description |
|--------|------|-------------|
| `HitUv` | `float2` | Normalized screen UV coordinates of the hit. |
| `Confidence` | `float` | Zero on a miss; fades near screen edges and the thickness limit. |

## Usage Example

```cpp
Texture2D FrameBufferCopy = /* .. */
Material m = /* .. */

float3 origin = /* world-space origin */;
float3 direction = /* world-space direction */;

TraceResult result = ScreenSpace::Trace( origin, direction );

if ( result.Confidence > 0.0f )
{
    float3 vReflectionColor = FrameBufferCopy.SampleLevel( g_sBilinearClamp, result.HitUv, 0 ).rgb;
    m.Emission = lerp( m.Emission, vReflectionColor, result.Confidence );
}
```
