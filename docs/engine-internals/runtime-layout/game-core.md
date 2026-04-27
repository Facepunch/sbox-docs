---
title: "Game Core"
icon: "inventory_2"
sources:
  - game/core/
updated: 2026-04-25
created: 2026-04-27
---

# Game Core

The `core` addon, located at `game/core/`, is the absolute foundation of the s&box runtime. 

## What is in `core`?

The `core` addon contains critical assets that you generally should not interact with directly, but which the engine uses constantly behind the scenes.

1. **Core Shaders:** The fundamental HLSL shader definitions required by the rendering pipeline (e.g., standard PBR, depth passes, shadow mapping). 
2. **Error Assets:** The iconic pink-and-black checkerboard textures and "ERROR" models used when an asset fails to load.
3. **Engine UI Resources:** Stylesheets, textures, and cursor icons used by the developer console, the profiling overlay, and engine-level dialogs.
4. **Configuration:** Default configuration files (`cfg`) for things like VR settings and internal engine cvars.
5. **Default Materials:** Basic unlit, standard, and debug materials.

As a game developer, you **rarely, if ever, need to reference `core` directly**. 

You do not need to add `core` to your project's dependency list in the `.sbproj` file; the engine intrinsically loads it before everything else.

However, if you are building deep editor tools or debugging rendering issues, you might find yourself inspecting the shaders or default materials located in this folder to understand how the engine expects data to be formatted.

## Troubleshooting

:::danger "Do not modify Core!"
Never edit files inside `game/core/`. If you change a fundamental shader or material here, your game will look different on your machine than it does on every player's machine, and your changes will be completely erased the next time s&box updates.
:::

:::warning "Pink and Black Textures"
If you see the textures from the `core` addon appearing on your models, it means your material path is invalid, the material failed to compile, or the file is missing from your mounted addons.
:::

## Related Pages
- [Base Addon](../../addons/base-code.md)
