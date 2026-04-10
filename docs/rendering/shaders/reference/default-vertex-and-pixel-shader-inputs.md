---
title: "Default Vertex and Pixel Shader Inputs"
icon: "🔏"
created: 2024-12-11
updated: 2024-12-12
---

# Default Vertex and Pixel Shader Inputs

If you're just opting for default Vertex and Pixel inputs which are usable on the default shading model, these are the reference member variables for each:

```cpp
struct VertexInput
{
    float3 vPositionOs : POSITION < Semantic( PosXyz ); >;
    float2 vTexCoord : TEXCOORD0 < Semantic( LowPrecisionUv ); >;	
    float4 vNormalOs : NORMAL < Semantic( OptionallyCompressedTangentFrame ); >;
    float4 vTangentUOs_flTangentVSign : TANGENT	< Semantic( TangentU_SignV ); >;
    
    //-------------------------------------------------------------------------------------------------------------------------------------------------------------
    // Skinning
    //-------------------------------------------------------------------------------------------------------------------------------------------------------------
    #if ( D_SKINNING > 0 )
    	uint4 vBlendIndices : BLENDINDICES 	< Semantic( BlendIndices ); >;
    	float4 vBlendWeight : BLENDWEIGHT 	< Semantic( BlendWeight ); >;
    #endif	
    	
    //-------------------------------------------------------------------------------------------------------------------------------------------------------------
    // SSS Curvature
    //-------------------------------------------------------------------------------------------------------------------------------------------------------------
    #if ( S_USE_PER_VERTEX_CURVATURE )
    	float flSSSCurvature : TEXCOORD2 < Semantic( Curvature ); >;
    #endif
    
    //-------------------------------------------------------------------------------------------------------------------------------------------------------------
    // Morph
    //-------------------------------------------------------------------------------------------------------------------------------------------------------------
    #if ( D_MORPH )
    	float nVertexIndex : TEXCOORD14 < Semantic( MorphIndex ); >;
    #endif
    
    //-------------------------------------------------------------------------------------------------------------------------------------------------------------
    // Instancing data
    //-------------------------------------------------------------------------------------------------------------------------------------------------------------
    uint nInstanceTransformID : TEXCOORD13 < Semantic( InstanceTransformUv ); >;
    
    //-------------------------------------------------------------------------------------------------------------------------------------------------------------
    // Baked lighting
    //-------------------------------------------------------------------------------------------------------------------------------------------------------------
    #if ( D_BAKED_LIGHTING_FROM_LIGHTMAP )	
    	float2 vLightmapUV : TEXCOORD3 < Semantic( LightmapUV ); > ;
    #endif
}
```


```cpp
struct PixelInput
{
    #if ( PROGRAM == VFX_PROGRAM_PS )
    	float3 vPositionWithOffsetWs : TEXCOORD0;
    #else
    	float3 vPositionWs : TEXCOORD0;
    #endif
    
    float3 vNormalWs 		: TEXCOORD1;
    float2 vTextureCoords 	: TEXCOORD2;
    float4 vVertexColor 	: TEXCOORD4;
    
    centroid float3 vCentroidNormalWs : TEXCOORD5; // For specular, used if interpolation sends normal outside the unit sphere

  	float3 vTangentUWs : TEXCOORD6; 
  	float3 vTangentVWs : TEXCOORD7; 
    
    #if ( S_USE_PER_VERTEX_CURVATURE )
    	float flSSSCurvature : TEXCOORD11;
    #endif
    
    #if ( D_BAKED_LIGHTING_FROM_LIGHTMAP )
    	centroid float2 vLightmapUV : TEXCOORD3;
    #endif
    
    //-------------------------------------------------------------------------------------------------------------------------------------------------------------
    // System interpolants
    //-------------------------------------------------------------------------------------------------------------------------------------------------------------
    
    #if ( PROGRAM != VFX_PROGRAM_PS ) 
    	float4 vPositionPs : SV_Position;
    #else // PS only
    	float4 vPositionSs : SV_Position;
    	#if ( S_RENDER_BACKFACES )
    		bool face : SV_IsFrontFace;
    	#endif
    #endif
}
```

Different combos expose different feature sets for these
