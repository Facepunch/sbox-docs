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

    // Gets the light structure given the world position, screen-space position
    // and the light index.
    static Light From( float3 vPositionWs, float4 vPositionSs, uint nLightIndex );
    
    // Number of lights in the current fragment.
    static uint Count( float4 vPositionSs );
};
```

### Iterating over all lights

```cpp
for( int i=0; i < Light::Count( ScreenPosition ); i++ )
{
    Light l = Light::From( WorldPosition, ScreenPosition, i );
    ...
}
```

This iterates both Dynamic and Static lights, this does all you need and returns with all optimizations from Frustum Tiled Lighting

# Binned Lights

This is the raw, internal information of lights that comes from the scene in the CPU, listed here for reference.

```cpp
class BinnedLight
{
    uint Type;          // 1 = point, 2 = directional, 3 = spot, 4 = rect
    LightShape Shape;   // Sphere, Capsule, Rectangle
    uint Flags;

    float4x4 LightToWorld;

    float3 Color;

    float LinearFalloff;
    float QuadraticFalloff;
    float FalloffBias;
    float Radius;
    float RadiusSquared;
    float2 ShapeSize;

    float4 SpotLightInnerOuterConeCosines; // x - inner cone, y - outer cone, z - reciprocal between inner and outer angle, w - Tangent of outer angle

    float FogIntensity;

    // Index in the shadow array, 0xFFFFFFFF if no shadow
    uint ShadowMapIndex;

    // Light Cookies, fancy image projection on light, 0xFFFFFFFF if no cookie
    uint LightCookieTextureIndex;

    // Custom shadow techniques precomputed on compute shader (RT, screenspace shadows, capsules, etc), 0xFFFFFFFF if none
    uint ShadowMaskTextureIndex;

	// ---------------------------------

	float3 GetPosition() 			{ return LightToWorld[3].xyz; }
	float3 GetDirection() 			{ return LightToWorld[0].xyz; }
	float3 GetDirectionUp() 		{ return LightToWorld[1].xyz; }

    bool IsSpotLight()              { return ( SpotLightInnerOuterConeCosines.x != 0.0f ); }

	// ---------------------------------

    bool IsDiffuseEnabled()             { return ( Flags & LightFlags::DiffuseEnabled ) != 0; }
    bool IsSpecularEnabled()            { return ( Flags & LightFlags::SpecularEnabled ) != 0; }
	bool IsTransmissiveEnabled() 	    { return ( Flags & LightFlags::TransmissiveEnabled ) != 0; }
    bool HasDynamicShadows() 	        { return ( ShadowMapIndex != 0xFFFFFFFF ); }
    bool HasLightCookie()               { return ( Flags & LightFlags::LightCookieEnabled ) != 0; }
    Texture2D GetLightCookieTexture()   { return Bindless::GetTexture2D( LightCookieTextureIndex ); }
};

StructuredBuffer<BinnedLight>    BinnedLightBufferV2    < Attribute( "BinnedLightBufferV2" );  > ;

BinnedLight DynamicLightConstantByIndex( int index )
{
    return BinnedLightBufferV2[ index ];
}

BinnedLight BakedIndexedLightConstantByIndex( int index )
{
    return BinnedLightBufferV2[ BakedLightIndexMapping[index].x ];
}
```
