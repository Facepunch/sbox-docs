---
title: "Screen Space Tracing"
icon: "🥢"
created: 2024-12-07
updated: 2024-12-09
---

# Screen Space Tracing

`ScreenSpace::Trace` provides functionality for tracing rays in screen space to compute effects like [Dynamic Reflections](/rendering/shaders/classes/dynamic-reflections.md) or any kind of ray in world space. It leverages hierarchical ray marching for efficient performance.

## API Reference

```csharp
static TraceResult Trace(
    const float3 Position,    // The world-space origin point of the trace.
    const float3 Direction,   // The world-space direction vector of the ray.
    const float2 vPositionSs, // Screen-space position corresponding to the origin.
    uint nMaxSteps = 64,      // Maximum steps for ray marching (default is 64).
);
```

The TraceResult structure represents the outcome of a trace operation:

* **HitClipSpace**  The hit position in clip space coordinates. With UV and Z Depth.
* **Confidence**  The confidence level of the hit detection.
* **ValidHit** Indicates whether a valid hit was found.

## Usage Example

```csharp
Texture2D FrameBufferCopy = /* .. */
Material m = /* .. */

float3 origin = /* world-space origin */;
float3 direction = /* world-space direction */;
float2 screenPos = /* screen-space position */;

TraceResult result = ScreenSpace::Trace(origin, direction, screenPos);

if (result.ValidHit)
{
    float3 vReflectionColor = FrameBufferCopy[ trace.HitClipSpace.xy ].rgb;
    m.Emission = lerp( m.Emission, vReflectionColor, result.Confidence );
}
```
