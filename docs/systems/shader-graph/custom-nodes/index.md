---
title: "Custom Nodes"
icon: "🆕"
created: 2025-08-20
updated: 2026-04-25
sources:
  - game/addons/tools/Code/ShaderGraph/
---

# Custom Nodes

You can expand Shader Graph by creating your own reusable nodes, either visually by collapsing logic into Subgraphs, or via code by writing C# Shader Nodes.

## Quick Working Example

You can define a custom Shader Graph node purely in C# that outputs raw HLSL code:

```csharp
using Sandbox;
using System;

[Title( "My Custom Math" ), Category( "Math" )]
public class MyMathNode : ShaderNode
{
	[Input] public float A { get; set; } = 1.0f;
	[Input] public float B { get; set; } = 2.0f;

	[Output] public float Result { get; set; }

	public override string GenerateCode()
	{
		// Inject raw HLSL directly into the compiled shader
		return $"{Result} = ({A} * {B}) + sin({A});";
	}
}
```

## Navigation Hub

* [**Creating a Subgraph**](creating-a-subgraph.md) - Learn how to select a group of existing visual nodes and collapse them into a reusable asset.
* [**C# Shader Nodes**](c-shader-nodes.md) - Learn how to use the `ShaderNode` class to inject custom HLSL code and create advanced mathematical nodes.

### When to use a Subgraph vs C# Node

* **Use a Subgraph** if you are an artist who wants to organize a messy graph, or if you want to reuse a specific visual effect (like a "scrolling fire noise" setup) across multiple different shaders.
* **Use a C# Shader Node** if you are a programmer who needs to execute a specific, highly optimized mathematical formula, or if you need to access specific HLSL intrinsic functions that aren't available as default visual nodes. 

## Troubleshooting

:::danger HLSL Syntax Errors
If you are writing a C# Shader Node, remember that the string returned by `GenerateCode()` is raw HLSL. If you miss a semicolon or use a type incorrectly, the entire Shader Graph will fail to compile. Check the output window in the Shader Graph editor for the exact HLSL error.
:::

## Related Pages
* [Action Resources](../../actiongraph/custom-nodes/action-resources.md) - The equivalent system for logic graphs.
