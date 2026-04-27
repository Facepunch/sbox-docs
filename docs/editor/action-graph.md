---
title: "ActionGraph"
icon: "🌲"
sources:
  - game/editor/ActionGraph/Code/
  - game/editor/ActionGraph/Code/EditorActionGraph.cs
updated: 2026-04-27
created: 2026-04-27
---

# ActionGraph

> **Heads up — this system is evolving.**
> *Action Graph might be deprecated in the future, and replaced by something else.*

ActionGraph is s&box's visual scripting system. This page covers the **editor UI**. For C# integration, custom nodes, and variables, see **[ActionGraph System](../systems/actiongraph/index.md)**.

## Using the Canvas

1. **Adding Nodes:** Right-click anywhere on the canvas to open the node palette. Search for C# methods, math operations, or Component functions.
2. **Connecting Nodes:** Drag between execution pins (arrow-shaped) to control logic flow. Drag between data pins (circles) to pass values.
3. **Variables:** Define graph-level variables via the Variables panel — accessible and modifiable by any node in the graph.

## Live Debugging

When playing in the editor with the graph window open, wires light up and display values as data passes through them. This lets you trace logic flow frame-by-frame without adding any extra code.

## Troubleshooting

:::warning
- **Graph not executing:** ActionGraphs don't auto-run — something must explicitly invoke them. Check that the Component or system responsible for triggering the graph is actually calling it.
- **Infinite Loops:** The engine detects and halts infinite loops (e.g. an output pin looped back to a prior node), but execution will stop and a warning is printed to console.
- **Missing Node errors:** If you rename or change the signature of a C# method exposed to the graph, existing nodes break. Right-click the broken node to fix or replace it.
:::

## Related Pages
- [ActionGraph System](../systems/actiongraph/index.md)
- [Using ActionGraph with C#](../systems/actiongraph/using-with-c.md)
- [Custom Nodes](../systems/actiongraph/custom-nodes/index.md)
