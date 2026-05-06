---
title: "DecalDefinition"
icon: "📄"
created: 2025-06-26
updated: 2025-06-26
---

# DecalDefinition

A DecalDefinition holds what a single Decal should look like. It describes its color, normals, roughness, metalness and ambient occlusion. If you define a Height texture, it will define its parallax too.


![](./images/decaldefinition.png)



| Parameter | Meaning |
|-----------|---------|
| Tint      | Tint the color of the decal. |
| Color Mix | If 1 the Color texture will be full alpha, if 0 it will be invisible. Useful if you just want normals. |
| Width, Height | The size the decal should appear in the world by default. |
