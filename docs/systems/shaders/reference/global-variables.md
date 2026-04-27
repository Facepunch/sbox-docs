---
title: "Global Variables"
icon: "🔢"
created: 2023-11-26
updated: 2023-11-26
sources:
  - game/addons/base/Assets/shaders/common/
---

# Global Variables

The shader compiler injects a set of constants into every shader's global namespace. They cover camera state, viewport geometry, time, fog flags, render-state hints, and tonemap. You don't need to declare or `#include` anything — they're always there.

The names follow Source 2's Hungarian-style prefixes:

| Prefix | Type |
|---|---|
| `g_fl` | scalar float |
| `g_v` | vector (`float2`/`float3`/`float4`) |
| `g_mat` | matrix (`float4x4`) |
| `g_n` | int |
| `g_b` | bool |

:::info Naming
These names are part of the engine's stable surface. There's been talk of nicer aliases (e.g. `time` instead of `g_flTime`) but for now use the originals.
:::

## Time

| Type | Name | Description |
|---|---|---|
| `float` | `g_flTime` | Current time in seconds. |

## Camera & viewport

| Type | Name | Description |
|---|---|---|
| `float4x4` | `g_matWorldToProjection` | World → projection (clip) space. |
| `float4x4` | `g_matProjectionToWorld` | Projection → world space (inverse of the above). |
| `float4x4` | `g_matWorldToView` | World → view (camera) space. |
| `float4x4` | `g_matViewToProjection` | View → projection space. |
| `float4x4` | `g_matProjectionToView` | Projection → view space. Used by `Depth::Linearize`. |
| `float3` | `g_vCameraPositionWs` | Camera world-space position. |
| `float3` | `g_vCameraDirWs` | Camera forward direction in world space. |
| `float3` | `g_vHighPrecisionLightingOffsetWs` | Offset added to vertex world-positions before lighting; helps reduce floating-point error when far from the origin. |
| `float` | `g_flViewportMinZ` | Near plane (in projected space). |
| `float` | `g_flViewportMaxZ` | Far plane. |
| `float2` | `g_vViewportSize` | Viewport size in pixels. |
| `float2` | `g_vInvViewportSize` | `1.0 / g_vViewportSize`. Multiply screen-pixel coordinates by this to get UV. |
| `float2` | `g_vViewportOffset` | Top-left of the viewport in render-target space (matters when the viewport is a sub-region of a larger target). |
| `float2` | `g_vRenderTargetSize` | Render-target size in pixels (may be larger than the viewport). |

## Fog flags

Set by the engine when fog volumes are active in the scene. Use these to gate fog work in custom shading models — calling `Fog::Apply` is fine in either case, but the flags let you skip cost when no fog is configured.

| Type | Name | Description |
|---|---|---|
| `bool` | `g_bFogEnabled` | Master fog gate; if false, all of the below are no-ops. |
| `bool` | `g_bGradientFogEnabled` | Gradient fog is active. |
| `bool` | `g_bCubemapFogEnabled` | Cubemap fog is active. |
| `bool` | `g_bVolumetricFogEnabled` | Volumetric fog is active. |

## Render state

| Type | Name | Description |
|---|---|---|
| `int` | `g_nMSAASampleCount` | Sample count for the bound render target (`1` = no MSAA). |
| `float` | `g_flAlphaTestReference` | Alpha-test cutoff used by `AdjustOpacityForAlphaToCoverage`. |
| `float` | `g_flAntiAliasedEdgeStrength` | Edge AA strength for alpha-to-coverage. |
| `bool` | `g_bWireframeMode` | True when the editor's wireframe overlay is on. |
| `float3` | `g_vWireframeColor` | Colour to draw with when `g_bWireframeMode` is true. |

## Tonemap

| Type | Name | Description |
|---|---|---|
| `float` | `g_flToneMapScalarLinear` | Linear tonemap scalar applied by the engine after shading. Used by `ToolsVis::*` overlays so debug colours match exposure. |

## Material common inputs

These are declared at the top of every material shader (via `Material.CommonInputs.hlsl`) so they show up in the Material Editor automatically:

| Type | Name | Description |
|---|---|---|
| `float3` | `g_flTintColor` | Per-material tint, multiplied by vertex colour. |
| `float` | `g_flSelfIllumScale` | Self-illumination scale, `0..16`. |

## Bindless texture / sampler arrays

Declared by `common/classes/Bindless.hlsl`. **Don't access these directly** — use `Bindless::GetTexture2D( idx )` etc. (see [Bindless API](../classes/bindless-api.md)).

| Type | Name | Notes |
|---|---|---|
| `Texture2D[]` | `g_bindless_Texture2D` | Use via `Bindless::GetTexture2D`. |
| `Texture2DMS<float4>[]` | `g_bindless_Texture2DMS` | Use via `Bindless::GetTexture2DMS`. |
| `Texture3D[]` | `g_bindless_Texture3D` | Use via `Bindless::GetTexture3D`. |
| `TextureCube[]` | `g_bindless_TextureCube` | Use via `Bindless::GetTextureCube`. |
| `Texture2DArray[]` | `g_bindless_Texture2DArray` | Use via `Bindless::GetTexture2DArray`. |
| `TextureCubeArray[]` | `g_bindless_TextureCubeArray` | Use via `Bindless::GetTextureCubeArray`. |
| `SamplerState[2048]` | `g_bindless_Sampler` | Use via `Bindless::GetSampler`. |
| `SamplerComparisonState[2048]` | `g_bindless_SamplerComparison` | Use via `Bindless::GetSamplerComparison`. |
| `RWTexture2D<float4>[]` | `g_bindless_RWTexture2D` | Compute shaders only. Use via `Bindless::GetRWTexture2D`. |
| `RWTexture3D<float4>[]` | `g_bindless_RWTexture3D` | Compute shaders only. |
| `RWTexture2DArray<float4>[]` | `g_bindless_RWTexture2DArray` | Compute shaders only. |

## See also

- [Default Vertex/Pixel Shader Inputs](default-vertex-and-pixel-shader-inputs.md) — what's in `VertexInput` / `PixelInput` per stage.
- [Vertex Helpers](vertex-helpers.md) — `ProcessVertex` / `FinalizeVertex`.
- [Bindless API](../classes/bindless-api.md) — wrappers for the bindless arrays above.
