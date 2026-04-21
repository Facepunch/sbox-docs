---
title: "Global Variables"
icon: "🔢"
created: 2023-11-26
updated: 2023-11-26
---

# Global Variables

The global namespace has a bunch of useful variables.



:::info
I don't love these names but they're not going to go anywhere, so you're free to use them. In all likelihood in the future we'll get together and create a bunch of nicer #define's for them, so `g_flTime` will be accessible via`Time.Now`, or just `time` etc.

:::


# Time

| Type | Name | Description |
|------|------|-------------|
| `float` | `g_flTime` | Current time |


# Camera

| Type | Name | Description |
|------|------|-------------|
| `float4x4` | `g_matWorldToProjection` |             |
| `float4x4` | `g_matProjectionToWorld` |             |
| `float4x4` | `g_matWorldToView` |             |
| `float4x4` | `g_matViewToProjection` |             |
| `float3` | `g_vCameraPositionWs` | Camera position in world space |
| `float3` | `g_vCameraDirWs` | Camera direction in world space |
| `float` | `g_flViewportMinZ` |             |
| `float` | `g_flViewportMaxZ` |             |
| `float2` | `g_vViewportSize` |             |
| `float2` | `g_vViewportOffset` |             |
| `float2` | `g_vRenderTargetSize` |             |
