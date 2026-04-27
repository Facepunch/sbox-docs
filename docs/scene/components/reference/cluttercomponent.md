---
title: "Clutter Component"
icon: "🌲"
sources:
  - engine/Sandbox.Engine/Scene/Components/Clutter/ClutterComponent.cs
created: 2026-04-25
updated: 2026-04-27
---

# Clutter Component

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. For the full setup guide — covering `ClutterDefinition`, Scatterer types, the paint tool, and GPU instancing — see **[Clutter System](../../../systems/clutter/index.md)**.

Scatters large amounts of small objects (grass, rocks, debris) across a scene efficiently using GPU-instanced rendering.

## Properties

| Property | Description |
|---|---|
| `Clutter` | The `ClutterDefinition` asset dictating what models to scatter and their weights/density. |
| `Seed` | Random seed for generation. Changing this produces different placements. |
| `Mode` | `Volume` — scatters within a fixed bounding box. `Infinite` — streams tiles around the camera. |

## Related Pages
- [Clutter System Guide](../../../systems/clutter/index.md)
- [Built-in Components](index.md)
