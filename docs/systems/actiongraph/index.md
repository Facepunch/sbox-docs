---
title: "ActionGraph"
icon: "🍝"
created: 2024-01-08
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Systems/ActionGraphs/ActionGraphResource.cs
  - engine/Sandbox.System/Attributes/ActionGraphs.cs
---

# ActionGraph

> **Heads up — this system is evolving.** Some node forms inside ActionGraph are already marked `[Obsolete]` in the engine (notably the standalone "constant" nodes — the engine prefers the properties panel for inline values now), and Facepunch has signalled they may consolidate visual scripting under a broader system in future. ActionGraph as a whole is **not** marked obsolete and is fully supported today; treat this as a "migrate when a clear successor ships" signal, not a stop sign.

ActionGraph is a visual scripting system that lets you build game logic using nodes and wires instead of writing C# code.

## Quick Start

You can quickly create a visual script for any Component without writing code.

1. Create an empty GameObject in the Scene and select it.
2. In the Inspector, click **Add Component**.
3. At the bottom of the list, choose **New ActionGraph Component...**
4. Name your component (e.g., `MyVisualLogic`) and save it.
5. The ActionGraph editor will open automatically, ready for you to add nodes!

![An example ActionGraph that implements jumping and falling with gravity](./images/an-example-actiongraph-that-implements-jumping-and-falling-w.png)

## Common Patterns
1. **Component Wrapping:** Write heavy logic in C# and expose simple wrapper methods to ActionGraph.
2. **Signal Routing:** Use ActionGraph to route signals from UI events to specific gameplay actions without hardcoding references.
3. **Asset Driven Logic:** Save complex graph behaviors as `.actiongraph` assets and load them dynamically on different objects at runtime.

## Navigation Hub

*   [**Intro to ActionGraphs**](intro-to-actiongraphs.md) - Learn the basics of nodes, links, and signals.
*   [**Variables**](variables.md) - How to store and retrieve data across your graph.
*   [**Using with C#**](using-with-c.md) - How to bridge ActionGraphs with your existing C# code.
*   [**Component Actions**](component-actions.md) - Build custom actions directly on your C# components.
*   [**Custom Nodes**](custom-nodes/index.md) - Create reusable nodes visually or via C# for others to use.
*   [**Examples**](examples.md) - Learn from practical ActionGraph patterns.

## Performance

*   **Loops in Visual Scripting:** Avoid performing massive `For Each` loops inside ActionGraphs that run every frame. If you need to iterate over hundreds of game objects, it is significantly faster (and easier to read) to write a C# Component and call it as a custom node from your graph.
*   **Component Lookups:** Try to cache references to specific GameObjects or Components using variables, rather than using generic "Find Component" nodes every frame.

*   **Hotloading:** Changes made to an ActionGraph are hotloaded instantly upon saving the `.action` file, just like C#. However, if the graph contains errors, the engine will halt the execution of that specific graph until fixed.

## Troubleshooting

:::warning My graph isn't doing anything!
Ensure your ActionGraph is actually being triggered. If you are using a Component ActionGraph, make sure the GameObject is enabled in the scene. Check your starting execution signal (usually an event like `On Update` or `On Start`). If the white signal wire is broken, the nodes won't run.
:::

## Sample Projects

*   [**Platformer Starter**](../../build-games/samples-templates/index.md) - Contains examples of player movement and item pickups entirely using ActionGraph.

## Related Pages
* [Code Basics](../../code/index.md)
