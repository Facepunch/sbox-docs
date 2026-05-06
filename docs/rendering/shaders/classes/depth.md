---
title: "Depth"
icon: "👀"
created: 2023-11-26
updated: 2024-12-09
---

# Depth

We have some utility functions for reading from the depth buffer. We generate it in an early depth pass, so you can re-use this even on operations that also write to depth.


:::info
s&box uses reverse-z depth as of July 2024 - this is the same as Unreal and Godot where 0 is far and 1 is near, this offers much greater precision.

:::


:::info
The depth texture is a Depth Mip-chain ( Also known as a Depth Pyramid ), it means the mipmaps of the texture store the minimum/maximum distance of the pixels in two color channels that are up on the chain (2x2, 4x4, 8x8 and so on), we use this to cull things in a GPU-driven manner and to accelerate screen-space traces.

:::

Depth can be separated in 3 spaces, Normalized and Linear depth are the ones you probably care about:

* **Raw:** The raw value of the depth buffer
* **Normalized ( Projected Space ):** Depth from 0..1 based on the z-near and z-far of the current viewport.
* **Linear ( View Space ):** Depth in world units from the camera

# API Reference

## Depth::Get( float2 ss )

Returns the distance on this screen position.

## Depth::Normalize( float d )

Given a distance (like the one returned from Depth::Get), it'll convert it to a float between 0-1 representing the distance between zfar and znear.

## Depth::GetNormalized( float2 ss )

Very much like Depth::Get, except the result is already normalized between 0 and 1, where 0 is far away and 1 is near.

## Depth::GetWorldPosition( float2 ss )

Return the position in world coordinates of the passed in screenspace position.

## Depth::Linearize( float d )

* Converts a raw value of the depth buffer into one in view space, with world units of distance away from the camera

## Depth::GetLinear( float2 ss )

* Given the position of a pixel, return the position in view space, with world units of distance away from the camera.

# Example Usage

On a translucent material, you could calculate the distance between the surface and what's behind the object, for effects like fog:

```csharp
Material m = ..

float3 fogColor = float3( 0.1f, 0.1f, 0.1f );
float fogRange = 100.0f;

float3 surfacePos = m.WorldPosition;
float3 depthPos   = Depth::GetWorldPosition( m.ScreenPosition );

float dist = distance( surfacePos, depthPos );

m.Emission = lerp( m.Emission, fogColor, saturate( dist / fogRange ) );
```
