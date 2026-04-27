---
title: "Motion"
icon: "🚝"
created: 2024-12-08
updated: 2024-12-09
---

# Motion

The `Motion` class provides functions for calculating motion vectors, reprojection, and performing temporal filtering in HLSL shaders. It is useful for implementing effects like Temporal Anti-Aliasing (TAA), motion blur or general reprojection by blending current and previous frame data.

> ⚠️ Right now this is limited to camera motion only, moving objects currently do not react to their own motion.

# API

## *float3* Motion::GetFromWorldPosition(*float3* worldPosition)

* Reprojects a world-space position to screen-space ( Depth in Z ) coordinates using the last frame's depth buffer.

## *float3* Motion::Get(*float2* screenPosition)

* Returns the position in clip space ( Pixel Position + Depth in Z ) of where this pixel was projected on the previous frame.

## *float4* Motion::TemporalFilter( *uint2* vPositionSs, *Texture2D* tCurrent, *Texture2D* tPrev, *float* flBlendWeight = 0.9f )

* A traditional Temporal Anti-Aliasing filter, takes the position of the current pixel and the texture of the current frame, and previous frame and using the motion vectors to blend them.
* It works as follows:
  * **Calculate Previous Frame UV Coordinates:**
    * Uses motion vectors to find where the current pixel was in the previous frame.
    * Converts screen-space position to UV coordinates for texture sampling.
  * **Compute Min and Max Sample Values:**
    * Iterates over a 3x3 pixel neighborhood around the current pixel.
    * Finds the minimum and maximum color values to clamp the previous frame's sample.
  * **Sample Previous Frame Texture:**
    * Samples the previous frame's texture at the calculated UV coordinates using bilinear filtering.
  * **Clamp Previous Frame Sample:**
    * Ensures the previous frame's data does not introduce artifacts when blending using a bounding box of colors.
  * **Blend Samples:**
    * Blends the current frame's sample with the clamped previous frame's sample based on the blend weight.
