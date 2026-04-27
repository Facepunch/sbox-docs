---
title: "Custom Nodes"
icon: "🆕"
created: 2024-01-09
updated: 2026-04-11
sources:
  - engine/Sandbox.System/Attributes/ActionGraphs.cs
  - engine/Sandbox.Engine/Systems/ActionGraphs/ActionGraphResource.cs
---

# Custom Nodes

You can expand the ActionGraph system by creating your own reusable custom nodes, either visually using Action Resources or via code using C# Method Nodes.

## Quick Start

You can create a custom node visually by selecting existing nodes and right-clicking and selecting **Create Custom Node...**

![Creating a new custom node from an existing graph.](./images/creating-a-new-custom-node-from-an-existing-graph.png)

Or you can create one in code by adding an attribute to a static method:

```csharp
/// <summary>
/// Increments the value by 1.
/// </summary>
[ActionGraphNode( "example.addone" ), Pure]
[Title( "Add One" ), Group( "Examples" ), Icon( "exposure_plus_1" )]
public static int AddOne( int value )
{
	return value + 1;
}
```

## Common Patterns
1. **Pure Nodes:** Use the `[Pure]` attribute for nodes that only calculate values and have no side effects (like adding two numbers). They won't have execution pins.
2. **Async Nodes:** You can create nodes that yield execution (like `Task.Delay`) by returning `async Task`.
3. **Contextual Nodes:** Use `this` extension methods to make nodes appear only when dragging off a specific type of pin (e.g., `public static void Jump(this Player myPlayer)`).

## Usage Approaches

There are two main ways to create custom nodes, depending on your needs:

### 1. Visual Scripting (Action Resources)
If you've built a useful sequence of nodes in the ActionGraph editor and want to reuse it, you can collapse it into an **Action Resource**. This creates a `.action` file that acts as a new node type in your project. It's great for reusing visual logic without writing any code.

👉 [**Learn how to create Action Resources**](action-resources.md)

### 2. Code (C# Method Nodes)
If you have complex logic that is difficult to express visually, or you want to expose existing C# systems to the ActionGraph, you can create a **C# Method Node**. By adding the `[ActionGraphNode]` attribute to any public static C# method, it instantly becomes available as a node in the graph editor.

👉 [**Learn how to create C# Method Nodes**](c-method-nodes.md)

## Troubleshooting

:::warning Choosing the Right Method
If a piece of logic involves a lot of math or complex loops, it is usually much cleaner and faster to write it in C# as a **C# Method Node** rather than trying to build a massive spaghetti graph using an **Action Resource**.
:::

## Related Pages
* [Intro to ActionGraphs](../intro-to-actiongraphs.md)
