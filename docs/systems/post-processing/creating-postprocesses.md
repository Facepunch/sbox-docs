---
title: "Creating PostProcesses"
icon: "🔧"
created: 2024-05-08
updated: 2025-10-02
sources:
  - engine/Sandbox.Engine/Scene/Components/PostProcessing/BasePostProcess.cs
---

# Creating PostProcesses

You can create custom post-processing effects by writing a C# Component that inherits from `BasePostProcess<T>` and a corresponding shader to modify the camera's output.

## Quick Working Example

```csharp
public sealed class MyBrightnessEffect : BasePostProcess<MyBrightnessEffect>
{
	[Property, Range( -1, 1 )]
	public float Brightness { get; set; } = 0.0f;

	public override void Render()
	{
		// 1. Get the blended value based on all active PostProcessVolumes
		float brightness = GetWeighted( x => x.Brightness );
		
		// Optimization: if the effect is 0, don't waste time drawing it
		if ( brightness.AlmostEqual( 0.0f ) ) return;

		// 2. Set the value to be used in the shader
		Attributes.Set( "brightness", 1 + brightness );

		// 3. Load the shader
		var shader = Material.FromShader( "shaders/postprocess/brightness.shader" );
		
		// 4. Create the blit command (copying the backbuffer first)
		var blit = BlitMode.WithBackbuffer( shader, Stage.AfterPostProcess, 200, false );
		
		// 5. Execute the render command
		Blit( blit, "Brightness" );
	}
}
```

### Getting Weighted Values

When you derive from `BasePostProcess<T>`, you should use `GetWeighted()` instead of accessing your properties directly in `Render()`. 

Why? Because your effect could be attached to a local `PostProcessVolume` that the camera is only *halfway* inside of. `GetWeighted` automatically calculates the correct blended value depending on the camera's position within any active volumes.

```csharp
// WRONG: Ignores volume blending entirely
// float strength = this.Intensity; 

// CORRECT: Smoothly blends depending on the camera's position in PostProcessVolumes
float strength = GetWeighted( x => x.Intensity );
```

### Simple Blits vs Backbuffer Blits

The `Render()` method requires you to create a `BlitMode` which tells the engine how to draw your shader.

- **`BlitMode.Simple`**: Draws your material directly. Fast, but you cannot sample the existing screen colors.
- **`BlitMode.WithBackbuffer`**: Copies the current screen into a texture called `ColorBuffer` before drawing. You **must** use this if your shader needs to sample the existing screen pixels (like adjusting brightness or blurring).

```csharp
// Simple draw
var simpleBlit = BlitMode.Simple( myShader, Stage.AfterPostProcess );

// Draw with access to the previous screen pixels
var backbufferBlit = BlitMode.WithBackbuffer( myShader, Stage.AfterPostProcess );
```

### The Shader File

Your HLSL shader file needs to sample the `ColorBuffer` attribute (created by `WithBackbuffer`) and apply your custom math.

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

    // Grab the screen texture passed by BlitMode.WithBackbuffer
    Texture2D colorBuffer < Attribute( "ColorBuffer" ); SrgbRead( true ); >;
    
    // Grab the property set in C# via Attributes.Set()
    float brightness < Attribute("brightness"); >;

    float4 MainPs( PixelInput i ) : SV_Target0
    {
        float2 uv = CalculateViewportUv( i.uv.xy );
        
        // Sample the original pixel color
        float4 color = colorBuffer.SampleLevel( g_sBilinearMirror, uv, 0 );

        // Apply our effect
        color.rgb *= brightness;

        return color;
    }
}
```

## Configuration

When returning a `BlitMode`, you can define the following settings:

| Parameter | Description |
|---|---|
| `Material` | The material/shader to use for the blit. |
| `RenderStage` | Where to place this in the render pipeline (e.g., `Stage.AfterPostProcess`). |
| `Order` | The order within the stage. Lower numbers get rendered first. |
| `WantsBackbuffer` | If true, the screen is copied to a texture called `ColorBuffer` before the blit. |

## Troubleshooting

:::danger My effect is solid black or pink!
If you are using `colorBuffer.SampleLevel()` in your shader but you forgot to use `BlitMode.WithBackbuffer` in your C# code, the `ColorBuffer` texture won't exist, leading to rendering errors.
:::

:::warning My effect snaps on/off instantly when entering a volume!
Ensure you are using `GetWeighted( x => x.MyProperty )` in your `Render()` method instead of reading the property directly off the component. If you read the property directly, you bypass the smooth distance blending provided by `PostProcessVolume`.
:::

## Related Pages
* [Post Process Volume](postprocessvolume.md)
* [Shaders](../../systems/shaders/index.md)