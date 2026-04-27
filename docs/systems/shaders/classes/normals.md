---
title: "Normals & Roughness"
icon: "📐"
created: 2026-04-25
updated: 2026-04-25
sources:
  - game/addons/base/Assets/shaders/common/classes/Normals.hlsl
---

# Normals & Roughness

`Normals::Sample` and `Roughness::Sample` read world-space normals and roughness from the depth-normals G-Buffer. They're what you call when you need to know about the surface at a screen pixel without recomputing it from scratch.

If the G-buffer doesn't have a normal at the requested pixel (the object didn't write to depth-normals), `Normals::Sample` falls back to reconstructing the normal from the depth buffer using `SampleFromDepth` — finite differences across the depth pyramid. The fallback is always available, so this works on every fragment.

```cpp
#include "common/classes/Normals.hlsl"
```

## API

```cpp
static float3 Normals::Sample( int2 screenPos, uint msaaSampleIndex = 0 );
static float3 Normals::SampleFromDepth( int2 screenPos );

static float  Roughness::Sample( int2 screenPos, uint msaaSampleIndex = 0 );
```

| Method | Returns |
|---|---|
| `Normals::Sample` | World-space normal in `-1..1` range. Reads from `DepthNormals` if available; reconstructs from depth otherwise. |
| `Normals::SampleFromDepth` | World-space normal reconstructed from depth via finite-difference cross product. Use directly if you want to bypass the G-Buffer. |
| `Roughness::Sample` | Roughness `0..1`, packed in the alpha channel of the same `DepthNormals` texture. |

The optional `msaaSampleIndex` lets MSAA shaders pick a specific sub-sample.

## Usage

```cpp
#include "common/classes/Normals.hlsl"

// In a post-process or screen-space-effect pixel shader:
float3 n = Normals::Sample( int2( i.vPositionSs.xy ) );
float r = Roughness::Sample( int2( i.vPositionSs.xy ) );

// ... use n / r however you need
```

## Notes

- The normals are stored in the `DepthNormals` G-Buffer texture, indexed by `NormalsTextureIndex`. If the index is `-1`, every call falls back to depth reconstruction — that's the case for shaders that don't enable the depth-normals pass.
- The depth-reconstruction fallback uses `Depth::GetWorldPosition` at three pixels (centre, +1 X, +1 Y) and crosses the deltas. Surface discontinuities can cause artefacts; for high-quality work, enable depth-normals on the source materials.
- The same `DepthNormals` texture stores roughness in alpha, so `Roughness::Sample` and `Normals::Sample` reading from the same pixel cost one texture fetch each (no cross-contamination).

## Related

- [Depth](depth.md) — the depth-buffer access used by the reconstruction fallback.
- [G-Buffer](g-buffer.md) — the higher-level "what's in the G-Buffer and how to use it" overview.
- [Screen Space Tracing](screen-space-tracing.md) — common consumer of these normals.
