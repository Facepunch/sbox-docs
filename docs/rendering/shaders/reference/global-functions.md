---
title: "Global Functions"
icon: "🔨"
created: 2026-08-31
updated: 2026-09-03
---

Engine comes with a bunch of helper functions that are accessible from every shader. Here's a list of them. Please note that this page omits a bunch of global functions that are tied to specific classes, for example Depth, G-Buffer and others - we have dedicated pages for them, so take a look at the "Classes" category.

Please note that everything here is provided "as is". We don't provide any guarantee that everything on this page will be always functional. Some of the functions here are pretty obscure and aren't used by most of the shaders, they exist here only for reference purposes. 

## Normal Transformations

```cpp
// Convert the packed normal (usually it's a texture) from [0, 1] range to [-1, 1]
// It is essentially "vPackedNormal * 2 - 1" + normalize
float3 DecodeNormal( float3 vPackedNormal )
```

```cpp
// Converts normals from tangent space to global space
float3 TransformNormal( float3 vNormalTs, float3 vGeometricNormalWs, float3 vTangentUWs, float3 vTangentVWs )
```

```cpp
// Converts normals from world space to tangent space
float3 NormalWorldToTangent( float3 vNormalWs, float3 vTangentUWs, float3 vTangentVWs )
```

```cpp
// Compute normal from XY components only
float3 ComputeNormalFromXY( float2 vNormalXY )

// Same effect, but for packed normal texture with RG channels only
float3 ComputeNormalFromRGTexture( float2 vPackedNormal )
```

## Value Remapping

```cpp
// Remaps a value in the range [A, B] to [C, D]
float RemapVal( float flOldVal, float flOldMin, float flOldMax, float flNewMin, float flNewMax )
```

```cpp
// Remaps a value in the range [A, B] to [C, D]
// Value smaller than A is mapped to C, and value greater than B is mapped to D
float RemapValClamped( float flOldVal, float flOldMin, float flOldMax, float flNewMin, float flNewMax )
```

## Camera Helpers

```cpp
// Returns a world-space ray from camera to pixel position
float3 CalculateCameraToPositionRayWs( float3 vPositionWs )
```

```cpp
// Returns a normalized world-space direction vector from camera to pixel position
float3 CalculateCameraToPositionDirWs( float3 vPositionWs )
```

```cpp
// Returns a tangent-space ray from camera to pixel position
float3 CalculateCameraToPositionRayTs( float3 vPositionWs, float3 vTangentUWs, float3 vTangentVWs, float3 vNormalWs )
```

```cpp
// Returns a normalized tangent-space direction vector from camera to pixel position
float3 CalculateCameraToPositionDirTs( float3 vPositionWs, float3 vTangentUWs, float3 vTangentVWs, float3 vNormalWs )
```

```cpp
// Returns a normalized world-space reflected vector
float3 CalculateCameraReflectionDirWs( float3 vPositionWs, float3 vNormalWs )
```

```cpp
// Returns the distance between pixel and camera
float CalculateDistanceToCamera( float3 vPositionWs )
```

## Projection Transforms

These functions use a bunch of pre-defined projection matrices, see "Common Variables" for a full list of globally available matrices. Note: **Ws** stands for world-space, **Vs** for view space, **Ps** for projection space.  

### World -> View space transforms

```cpp
float4 Position4WsToVs( float4 vPositionWs )
```

```cpp
float4 Position3WsToVs( float3 vPositionWs )
```

```cpp
float3 Vector3WsToVs( float3 vVecWs )
```

```cpp
float3 Vector3VsToWs( float3 vVecVs )
```

### View -> Projection space

```cpp
float4 Position4VsToPs( float4 vPositionVs )
```

```cpp
float4 Position3VsToPs( float3 vPositionVs )
```

### World -> Projection space

```cpp
float4 Position4WsToPs( float4 vPositionWs )
```

```cpp
float4 Position3WsToPs( float3 vPositionWs )
```

## Viewport UV helpers

```cpp
// Calculates UV in [0, 1] for the given viewport size
float2 CalculateViewportUvFromInvSize( float2 vPositionSs, float2 vInvViewportSize )
```

```cpp
// Calculates UV in [0, 1] range for the current viewport  
float2 CalculateViewportUv( float2 vPositionSs )
```

## sRGB Gamma Conversions

```cpp
float3 SrgbGammaToLinear( float3 vSrgbGammaColor )
```

```cpp
float3 SrgbLinearToGamma( float3 vLinearColor )
```

## Color Manipulations

```cpp
// Computes grayscale luminance value from color input
float Luminance( float3 vColor )
```

```cpp
// Controls color saturation. 1 is the default color, 0 will turn the image gray, any values above 1 will start boosting color saturation
float3 SaturateColor( float3 vColor, float flSaturationAmount )
```

## RGB <-> HSV conversions

```cpp
float3 RgbToHsv( float3 vRbgColor )
```

```cpp
float3 HsvToRgb( float3 vHsvColor )
```

## Rotation Matrix Helpers

```cpp
// Build a matrix which rotates around an axis
float3x3 MatrixBuildRotationAboutAxisRadians( float3 vAxisOfRotation, float flRadians )
```

```cpp
// Build a matrix which rotates around an arbitrary axis.
float3x3 MatrixBuildRotationAboutAxis( float3 vAxisOfRotation, float flAngleDegrees )
```

```cpp
// Build a matrix from angles (pitch, yaw, roll)
// Angles values are expected to be in [0 to 360] range
float3x3 RotationMatrixFromAngles( float3 angles )
```

