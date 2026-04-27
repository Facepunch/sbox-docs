---
title: "Cable Component"
icon: "🔌"
sources:
  - engine/Sandbox.Engine/Scene/Components/Mesh/Cable/CableComponent.cs
  - engine/Sandbox.Engine/Scene/Components/Mesh/Cable/CableNodeComponent.cs
updated: 2026-04-25
created: 2026-04-27
---

# Cable Component

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `CableComponent` component.

The `CableComponent` allows you to procedurally generate and render a 3D non-destructive cable or rope mesh by connecting multiple control points.

## Quick Working Example

Usually, you build a cable in the Editor by adding `CableNodeComponent` GameObjects as children to your `CableComponent`. However, you can create one dynamically from code:

```csharp
using Sandbox;

public class CableSpawner : Component
{
    protected override void OnStart()
    {
        // 1. Create the main cable object
        var cableObj = new GameObject( true, "MyCable" );
        var cable = cableObj.Components.Create<CableComponent>();
        cable.Size = 4f;
        cable.Slack = 20f;

        // 2. Add nodes (control points) as children
        var node1 = new GameObject( true, "Node1" );
        node1.SetParent( cableObj );
        node1.LocalPosition = Vector3.Zero;
        node1.Components.Create<CableNodeComponent>();

        var node2 = new GameObject( true, "Node2" );
        node2.SetParent( cableObj );
        node2.LocalPosition = new Vector3( 100, 100, 0 );
        node2.Components.Create<CableNodeComponent>();

        // The cable mesh will automatically generate between the nodes
    }
}
```

### Adjusting Slack
A perfectly straight line rarely looks like a natural cable. You can increase the `Slack` property to make the cable artificially droop down between its nodes, simulating gravity.

### Node Customization
Each child `CableNodeComponent` has its own `RadiusScale` and `Roll` properties. This means you can make a cable thicker in the middle or twist the texture around a specific point by selecting individual nodes in the Scene Tree.

## Configuration

| Property | Description |
|---|---|
| `Size` | The base radius thickness of the cable mesh. |
| `Subdivisions` | The number of sides the tubular mesh has around its circumference (minimum 3). Higher values make it smoother but cost more performance. |
| `PathDetail` | How many extra smoothing segments to generate between each node. Higher values create smoother drooping curves. |
| `Slack` | How much the cable droops down between nodes. |
| `CapEnds` | If true, generates flat faces at the start and end of the cable to seal the tube. |
| `Material` | The material applied to the generated mesh. |
| `TextureOrientation` | Whether the material texture maps `Horizontal` or `Vertical` along the path. |
| `TextureScale` | Scales the material texture along the length of the cable. |

## Troubleshooting

:::warning No mesh is visible!
Ensure your `CableComponent` has at least **two** child GameObjects with `CableNodeComponent`s attached, and that they are positioned far enough apart. A cable cannot be drawn with only one point. Also verify that a valid `Material` is assigned.
:::

:::danger The cable looks jagged!
If your cable has sharp corners at the nodes instead of smooth curves, try increasing the `PathDetail` property. Also ensure your `Slack` is not set to an extreme value that forces unnatural bending.
:::

## Related Pages
* [Components Overview](../index.md)
