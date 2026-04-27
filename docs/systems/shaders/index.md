---
title: "Shaders"
icon: "🎨"
created: 2023-11-13
updated: 2025-06-15
---

# Shaders

The Shader system allows you to write custom rendering logic to control exactly how 3D models, screens, and compute operations are processed by the GPU.

## Quick Working Example

You can write shaders in HLSL. Here is a simple example of assigning a shader variable (Attribute) from C#.

```csharp
public class CustomShaderController : Component
{
	[Property] public ModelRenderer Renderer { get; set; }
	[Property] public Color GlowColor { get; set; } = Color.Red;

	protected override void OnUpdate()
	{
		if ( Renderer.IsValid() )
		{
			// Set a custom shader attribute on this specific object
			Renderer.SceneObject.Attributes.Set( "GlowColor", GlowColor );
		}
	}
}
```

## Navigation Hub

* [**Getting Started**](getting-started.md) - Learn how to set up your IDE and start writing HLSL shaders in s&box.
* [**Attributes and Variables**](attributes-and-variables.md) - How to pass data between your C# code, the Material Editor, and your GPU shaders.
* [**Classes**](classes/index.md) - Learn about the different rendering passes (like Depth, Forward).
* [**GPU Instancing**](gpu-instancing.md) - Draw thousands of identical meshes efficiently.
* [**Command Lists**](command-lists.md) - How to execute compute shaders and dispatch custom rendering commands.
* [**Modes**](modes.md) - Configure culling, blending, and depth testing modes.
* [**Shading Model**](shading-model.md) - Standard physically-based rendering (PBR) shading models.
* [**Material**](material.md) - Working with the base material properties.
* [**Render Attributes**](render-attributes.md) - Passing data directly from C# code to shaders.
* [**Render States**](render-states.md) - Configuration settings for the render pipeline.
* [**Sampler States**](sampler-states.md) - How to filter and sample textures in your shaders.
* [**Reference**](reference/index.md) - Documentation for built-in variables and helper functions.
