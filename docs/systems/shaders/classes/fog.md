---
title: "Fog"
icon: "🌁"
created: 2024-12-08
updated: 2024-12-08
---

# Fog

This class applies atmospheric effects to a fragment.

Only useful if you're doing your own [Shading Model](../shading-model.md), otherwise this is already all implied when you use `ShadingModelStandard`

# API

## *float3* Fog::Apply( *float3* worldPos, *float2* screenPos, *float3* color )

* Applies all atmospheric affects ( Gradient Fog, Cubemap Fog, Volumetric Fog ) to this fragment.
