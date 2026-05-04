---
title: "Texture Naming"
icon: "🖼️"
created: 2026-04-08
updated: 2026-04-08
---

# Texture Naming

When creating textures it's highly recommended to name them in a certain way so that they can be automatically populated in the material editor.

| **Texture Type**           | **Name**   |
| -------------------------- | ---------- |
| Base Color/Albedo/Diffuse  | _color     |
| Normal Map                 | _normal    |
| Roughness                  | _rough     |
| Metallic                   | _metal     |
| Ambient Occlusion          | _ao        |
| Opacity/Alpha/Transparency | _trans     |
| Emissive/Self Illumination | _selfillum |
| Tint Mask                  | _mask      |
| Blend Mask                 | _blend     |
| Height                     | _height    |
| Freedom of Motion          | _freedom   |

:::info
These names are configured by the shader, so a custom shader may require different naming.
:::