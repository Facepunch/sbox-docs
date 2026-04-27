---
title: "Clutter"
icon: "🍃"
sources:
  - engine/Sandbox.Engine/Scene/Components/Clutter/ClutterComponent.cs
  - engine/Sandbox.Engine/Resources/Clutter/ClutterDefinition.cs
updated: 2026-04-27
created: 2026-04-27
---

# Clutter

The Clutter System lets you quickly scatter large amounts of small objects like grass, rocks, and debris across your scene.

![](./images/clutter.png)

## Quick Working Example

You can add and configure a `ClutterComponent` directly via code, loading an existing Clutter Definition.

```csharp
using Sandbox;
using Sandbox.Clutter;

public sealed class ClutterSpawner : Component
{
    protected override void OnStart()
    {
        var clutterComponent = Components.GetOrCreate<ClutterComponent>();

        // Load an existing Clutter Definition asset
        clutterComponent.Clutter = ResourceLibrary.Get<ClutterDefinition>("path/to/my_clutter.clutter");

        // Set the mode to Infinite streaming around the camera
        clutterComponent.Mode = ClutterComponent.ClutterMode.Infinite;
        
        // Provide a random seed for variation
        clutterComponent.Seed = 12345;
    }
}
```

### Clutter Definition (Resource)

A `ClutterDefinition` is an asset that describes what to scatter. You create it in the Editor via **Create > Clutter Definition**.

*   **Entries:** A list of models or prefabs to scatter. Each entry has a **Weight** value that controls how likely it is to be picked relative to other entries. Use models for simple static props like grass or rocks, and prefabs when you need components or behaviors.
*   **Scatterer:** Controls how entries are distributed across the surface.
*   **Streaming:** Controls the tile grid used for generation.

### The Clutter Tool

Instead of relying purely on procedural generation, you can use the Clutter Tool in the editor toolbar to paint and erase clutter instances by hand.

*   **Paint:** Left-click to scatter instances at the brush location using the assigned scatterer.
*   **Erase:** Ctrl+click to remove instances within the brush radius.

Painted instances are stored separately from procedurally generated ones and persist with the scene.

## Configuration

### Clutter Component Mode

When configuring a `ClutterComponent`, the most important setting is its `Mode`:

| Mode | Description |
|---|---|
| **Infinite** | Streams tiles around the camera automatically. Tiles are created and destroyed as the camera moves. Use this for open-world ground cover. |
| **Volume** | Generates clutter within a fixed bounding box defined by a `VolumeComponent` (if attached) or the Clutter Component's own defined volume. Click **Generate** in the editor to populate. |

### Scatterer Types

Inside your `ClutterDefinition` asset, you can choose different Scatterers:

| Scatterer | Description |
|---|---|
| **Simple** | Random placement with configurable density, scale range, ground alignment, and height offset. Good for most use cases. |
| **Slope** | Filters entries by surface angle. Map different entries to different slope ranges (e.g., grass on flat ground, rocks on steep slopes). |
| **Terrain Material** | Places entries based on the terrain material at each point. Map specific entries to specific terrain layers (e.g., flowers on dirt, moss on rock). |

*(You can also implement your own custom scatterers by extending the `Scatterer` class in C#.)*

## Troubleshooting

:::danger "No clutter is showing up!"
If you are in **Volume** mode, you must click the **Generate** button in the inspector for the clutter to appear. If you are in **Infinite** mode, ensure there is an active `CameraComponent` in the scene, as it uses the camera's position to stream the clutter tiles.
:::

:::warning "Clutter is floating in the air!"
The Clutter system traces downward to place objects on the ground. Make sure the ground (whether it's a `Terrain` or a mesh with a `Collider`) has physics collision enabled. Without collision, the system doesn't know where the ground is.
:::

## Related Pages
- [Terrain](../terrain/index.md)
- [Assets](../../assets/index.md)