---
title: "Screen Space Tracing"
icon: "🥢"
created: 2024-12-07
updated: 2024-12-09
sources:
  - game/addons/base/Assets/shaders/common/classes/ScreenSpaceTrace.hlsl
---

# Screen Space Tracing

`ScreenSpace::Trace` ray-marches against the current depth buffer to compute screen-space hit data. It's the core primitive behind [Dynamic Reflections](dynamic-reflections.md), and you can use it directly for any custom screen-space ray (water surface tests, custom SSR variants, contact shadows, etc.).

## API Reference

```cpp
static TraceResult ScreenSpace::Trace(
    const float3 Position,    // World-space origin of the ray.
    const float3 Direction,   // World-space direction of the ray.
    uint nMaxSteps = 64       // Max ray-marching iterations.
);
```

The `TraceResult` struct:

* **HitClipSpace** — `float3`. Hit position in clip space. XY is pixel position (multiply by `g_vInvViewportSize` for UV). Z is depth.
* **Confidence** — `float`, `0..1`. Trace confidence — fade out the contribution by this rather than gating purely on `ValidHit`.
* **ValidHit** — `bool`. Whether the trace produced a hit.

Internally, `Trace` builds clip-space origin/direction from `g_matWorldToProjection`, then calls `HierarchicalRaymarch::Trace` against the depth pyramid. Depth thickness is adaptive (`sqrt(distanceToCamera)`) and back-tracing is on by default.

## Usage Example

```cpp
Texture2D FrameBufferCopy = /* prior pass output */;
Material m = /* shading state */;

float3 origin    = /* world-space origin */;
float3 direction = /* world-space direction */;

TraceResult result = ScreenSpace::Trace( origin, direction );

if ( result.ValidHit )
{
    float3 vReflectionColor = FrameBufferCopy[ result.HitClipSpace.xy ].rgb;
    m.Emission = lerp( m.Emission, vReflectionColor, result.Confidence );
}
```
