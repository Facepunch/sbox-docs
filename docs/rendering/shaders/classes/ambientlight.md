---
title: "AmbientLight"
icon: "🌥️"
created: 2024-12-19
updated: 2024-12-19
---

# AmbientLight

This class returns the ambient (diffuse) light reaching a point, automatically resolving whichever source is active for that position: an environment map probe, a baked lightmap probe volume, or DDGI (Dynamic Diffuse Global Illumination).

Only useful if you're doing your own [Shading Model](/rendering/shaders/shading-model.md), otherwise this is already handled when you use `ShadingModelStandard`.

# API

## *float3* AmbientLight::From( *float3* WorldPosition, *float4* PositionSs, *float3* WorldNormal )

* Returns the ambient diffuse light at the given world position and normal.
