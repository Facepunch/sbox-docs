---
title: "Creating PostProcesses"
icon: "🔧"
created: 2024-05-08
updated: 2025-10-02
---

# Creating PostProcesses

# The Component

To make a post process shader you should derive from `BasePostProcess<T>`. This makes it easier to make a component that will be able to blend from multiple others.

```csharp
public sealed class MyBrightnessEffect: BasePostProcess<MyBrightnessEffect>
{
	[Property, Range( -1, 1 )]
	public float Brightness{ get; set; } = 0.0f;

	public override void Render()
	{
		float brightness = GetWeighted( x => x.Brightness );
		if ( brightness.AlmostEqual( 0.0f ) ) return

		Attributes.Set( "brightness", 1 + brightness );

		var shader = Material.FromShader( "shaders/postprocess/brightness.shader" );
		var blit = BlitMode.WithBackbuffer( shader, Stage.AfterPostProcess, 200, false );
		Blit( blit, "Brightness" );
	}
}
```


There are a few things here that might need more detail.

### GetWeighted

This method gets a weighted value from all of the volumes we might be in. So instead of using Intensity from the component itself, we get it from this method.. which will look at all the active volumes and give us a blended, weighted value according to where the Camera is.

### Blit

This method will create a CommandList, which will later be executed when rendering. You can specify the render stage and the order.

You can also specify whether your effect needs the backbuffer or not. If you do, it'll be passed to your shader using the name`ColorBuffer`.

## Shader

Here's a very basic shader that will let the component change the brightness.

```cpp
COMMON
{
    #include "postprocess/shared.hlsl"
}

struct VertexInput
{
    float3 pos : POSITION < Semantic( PosXyz ); >;
    float2 uv : TEXCOORD0 < Semantic( LowPrecisionUv ); >;
};

struct PixelInput
{
    float2 uv : TEXCOORD0;
	float4 pos : SV_Position;
};

VS
{
    PixelInput MainVs( VertexInput i )
    {
        PixelInput o;
        
        o.pos = float4(i.pos.xy, 0.0f, 1.0f);
        o.uv = i.uv;
        return o;
    }
}

PS
{
    #include "postprocess/common.hlsl"
    #include "postprocess/functions.hlsl"
    #include "procedural.hlsl"

    Texture2D colorBuffer < Attribute( "ColorBuffer" ); SrgbRead( true ); >;
    float brightness< Attribute("brightness"); >;

    float4 MainPs( PixelInput i ) : SV_Target0
    {
        float2 uv = CalculateViewportUv( i.uv.xy );
        float4 color = colorBuffer.SampleLevel( g_sBilinearMirror, uv, 0 );

        color.rgb *= brightness;

        return color;
    }
}
```
