---
title: "Shader Graph"
icon: "📈"
created: 2025-08-20
updated: 2026-04-14
---

# Shader Graph

Shader Graph is a node-based visual scripting system that allows you to create custom shaders and materials without writing code.

## Quick Start

You can open the Shader Graph editor by creating a new asset:
1. Right-click in your Asset Browser.
2. Select **New -> Shader Graph**.
3. Double-click the new asset to open the editor.

## Navigation Hub

* [**Getting Started**](getting-started.md) - A guided tour of the editor, nodes, and inputs/outputs.
* [**Properties**](properties.md) - Configuring your shader's blend mode, shading model, and domain.
* [**Variables**](variables.md) - How to expose parameters to the Material Editor so artists can tweak them.
* [**Custom Nodes**](custom-nodes/index.md) - Advanced techniques for creating reusable subgraphs or writing C# shader nodes.

### Exposing Properties to Artists

If you want an artist to be able to change a value (like the color of a glow) without opening the Shader Graph, you use a **Variable**.

Any Constant node (like a Float or Color) that is given a name will automatically appear as an adjustable slider or color picker when someone creates a Material using your shader.

### Building Reusable Functions

If you find yourself creating the same complex web of 10 nodes over and over, you can collapse them into a **Subgraph**. This creates a single custom node that you can easily drop into any other shader, keeping your graphs clean and organized.

## Troubleshooting

:::warning My variables aren't showing up in the Material Editor!
Make sure you have given your Constant node a name in its properties. Unnamed constants are hardcoded into the shader and will not be exposed to the user.
:::

:::danger The preview is black or broken!
Check the **Output** tab at the bottom of the Shader Graph editor. This window displays compilation errors and warnings that will tell you exactly which node is missing an input or mathematically invalid.
:::

## Related Pages
* [ActionGraph](../actiongraph/index.md)
* [Shaders](../shaders/index.md)
