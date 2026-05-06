---
title: "Light"
icon: "💡"
created: 2024-12-08
updated: 2024-12-09
---

# Light

If you want to access lighting information directly you can use this.

# Lights

Lights can be easily accessed using this class, this already implies going over the surface of what's being drawn.

```cpp
struct Light
{
    // The color is an RGB value in the linear sRGB color space.
    float3 Color;

    // The normalized light vector, in world space (direction from the
    // current fragment's position to the light).
    float3 Direction;

    // The position of the light in world space. This value is the same as
    // Direction for directional lights.
    float3 Position;

    // Attenuation of the light based on the distance from the current
    // fragment to the light in world space. This value between 0.0 and 1.0
    // is computed differently for each type of light (it's always 1.0 for
    // directional lights).
    float Attenuation;

    // Visibility factor computed from shadow maps or other occlusion data
    // specific to the light being evaluated. This value is between 0.0 and
    // 1.0.
    float Visibility;

    // Gets the light structure given the screen-space and world position and
    // the light index.
    static Light From( float4 vPositionSs, float3 vPositionWs, uint nLightIndex );
    
    // Number of lights in the current fragment.
    static uint Count( float2 vPositionSs );
};
```

### Iterating over all lights

```cpp
for( int i=0; i < Light::Count(); i++ )
{
    Light l = Light::From( ScreenPosition, WorldPosition, i );
    ...
}
```

This iterates both Dynamic and Static lights, this does all you need and returns with all optimizations from Frustum Tiled Lighting

# Binned Lights

This is the raw, internal information of lights that comes from the scene in the CPU, listed here for reference.

```cpp
class BinnedLight
{
    uint4 Params;                           // x = num sequential frusta, y = cookie texture index, z = flags, w = light type
    float4 Color;                          // w - Linear radius
    float4 FalloffParams;                  // x - Linear falloff, y - quadratic falloff, z - radius squared for culling, w - truncation
    float4 SpotLightInnerOuterConeCosines; // x - inner cone, y - outer cone, z - reciprocal between inner and outer angle, w - Tangent of outer angle
    float4 Shape;                          // xy - size,  zw - unused

    float4x4 LightToWorld;

    // Shadow
    float4 ShadowBounds   [ MAX_SHADOW_FRUSTA_PER_LIGHT ];
    float4 ShadowVigniette[ MAX_SHADOW_FRUSTA_PER_LIGHT ];
    float4x4 WorldToShadow[ MAX_SHADOW_FRUSTA_PER_LIGHT ];
    float4 ProjectedShadowDepthToLinearDepth; // Assume all frusta have the same depth range

    float4x4 WorldToLightCookie; // Could just be world to light now, we are bindless! Still could use it for stuff like cloud layers if we keep separate though

	// ---------------------------------

    uint    NumShadowFrusta()      { return Params.x; }
    
	float3 GetPosition() 			{ return LightToWorld[3].xyz; }
	float3 GetDirection() 			{ return LightToWorld[0].xyz; }
	float3 GetDirectionUp() 		{ return LightToWorld[1].xyz; }
	float3 GetColor() 			    { return Color.xyz; }

	float GetLinearFalloff() 		{ return FalloffParams.x; }
	float GetQuadraticFalloff() 	{ return FalloffParams.y; }
	float GetRadiusSquared() 		{ return FalloffParams.z; }
	float GetRadius() 		        { return Color.w; }

	float2 GetShapeSize() 			{ return Shape.xy; }
	uint   GetType() 				{ return Params.z; }

    bool IsSpotLight()              { return ( SpotLightInnerOuterConeCosines.x != 0.0f ); }

	// ---------------------------------

    bool IsVisible()                    { return ( Params.z & LightFlags::Visible ) != 0; }
    bool IsDiffuseEnabled()             { return ( Params.z & LightFlags::DiffuseEnabled ) != 0; }
    bool IsSpecularEnabled()            { return ( Params.z & LightFlags::SpecularEnabled ) != 0; }
	bool IsTransmissiveEnabled() 	    { return ( Params.z & LightFlags::TransmissiveEnabled ) != 0; }
    bool HasDynamicShadows() 	        { return ( Params.z & LightFlags::ShadowEnabled ) != 0; }
    bool HasLightCookie()               { return ( Params.z & LightFlags::LightCookieEnabled ) != 0; }
    bool HasFrustumFeathering()         { return ( Params.z & LightFlags::FrustumFeathering ) != 0; }
    Texture2D GetLightCookieTexture()   { return g_bindless_Texture2D[ Params.y ]; }
};

StructuredBuffer<BinnedLight>    BinnedLightBuffer    < Attribute( "BinnedLightBuffer" );  > ;

BinnedLight DynamicLightConstantByIndex( int index )
{
    return BinnedLightBuffer[ index ];
}

BinnedLight BakedIndexedLightConstantByIndex( int index )
{
    return BinnedLightBuffer[ BakedLightIndexMapping[index].x ];
}
```
