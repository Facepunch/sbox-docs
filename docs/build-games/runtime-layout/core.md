---
title: Game Core & Editor Layout
icon: "🧠"
sources:
  - game/core
  - game/editor
created: 2026-04-27
updated: 2026-04-27
---

# Game Core & Editor Layout

![s&box editor](https://files.facepunch.com/matt/1b2211b1/sbox-dev_FoZ5NNZQTi.jpg)
*The Editor relies on core assets to function correctly.*

The `game/core/` and `game/editor/` directories contain the foundational assets, shaders, materials, and internal tools that the engine and development environment rely on to function.

## Quick Start Example

When you are writing a component, you will frequently reference assets located in the `core` directory. You do this without specifically stating "core" in the path, because `game/core` is mounted as the root of the virtual filesystem.

```csharp
using Sandbox;

public sealed class CoreAssetExample : Component
{
    protected override void OnStart()
    {
        // "materials/dev/dev_measuregeneric01b.vmat" physically lives in
        // game/core/materials/dev/... but is accessed seamlessly.
        var myMaterial = Material.Load( "materials/dev/dev_measuregeneric01b.vmat" );
    }
}
```

## Sandbox.Engine Mounting Interceptors & Layout Loading Sequence

The `Sandbox.Engine` library plays a crucial role in how the layout loading sequence operates. When the engine mounts an addon or a legacy game (like Half-Life 2), the mounting system doesn't just copy files. Instead, it registers **interceptors** within the virtual filesystem (VFS) based on precedence. 

The precise loading sequence happens step-by-step to construct the final VFS state your game code sees:

1.  **Core Initialization (`game/core`):** The engine boots and immediately mounts the base `core` addon. All baseline primitives (like `materials/dev/...` and core shaders) are populated into the VFS. This is the absolute fallback layer.
2.  **Editor Payload (`game/editor`):** If running in the development environment, editor assets and internal tools are mounted on top of `core`.
3.  **Active Addon/Game Mounting:** The game you are running (or building) is mounted. Any file in your addon that matches a path in `core` will take precedence over it (e.g. if you provide your own `materials/dev/error.vmat`).
4.  **Legacy Interceptor Registration:** Finally, `MountHost` looks for external or legacy mounts (like GoldSrc or Natural Selection 2). It registers specific file interceptors for custom extensions (e.g., `.wad`, `.model`, `.material`). These interceptors have dynamic priority.

During runtime, when your game code requests an asset via `FileSystem.OpenRead` (e.g., calling `Material.Load()`), `Sandbox.Engine` executes the resolution loop in reverse:
* **Interception Check:** The VFS checks the highest-priority registered interceptors. If a legacy mount handles `.wad` files, the interceptor parses the raw bytes and dynamically generates a modern Source 2 texture in-memory.
* **Addon Check:** If not intercepted, it checks the active addon's physical disk layout.
* **Core Check:** If still not found, it falls back to the `game/core` physical disk layout.

This architecture means the physical runtime layout on disk can differ significantly from what the game code sees. Addons can transparently override core files because their directory structure maps identically onto the VFS with higher precedence.

```csharp
// If your addon provides "materials/dev/primary_black_color.vtex",
// the VFS resolves this request to your addon's file first.
// If it fails or isn't present, it falls back to the core engine's file.
var texture = Texture.Load( "materials/dev/primary_black_color.vtex" );
```

## Common Patterns

1. **Root Mounting:** Because `game/core` is mounted globally, its contents (like `materials/`, `sounds/`, `shaders/`) define the base directory structure of the virtual filesystem. When you create an addon, your `materials/` folder merges with the core `materials/` folder.
2. **Developer Textures:** The `core/materials/dev/` folder contains the standard grid textures used for blockout and prototyping in Hammer.
3. **Core Shaders:** The underlying graphics pipeline is defined by the HLSL files in `core/shaders/`. If you create a custom ShaderGraph, it compiles down into code that relies on these core shading frameworks.

## Directory Breakdown

### `game/core/`
*   **`cfg/`:** Global engine configuration files and default keybinds.
*   **`materials/` & `textures/`:** Base engine materials (error materials, developer grids, missing textures).
*   **`shaders/`:** The core Source 2 shader files (`.vfx`) and include headers used to render everything.
*   **`sounds/`:** Default UI sounds and the engine error sound.
*   **`tools/`:** Internal scripts and definitions used by the compilation pipeline.

### `game/editor/`
*   **`ActionGraph/`:** Logic for the node-based visual scripting editor.
*   **`Hammer/`:** Extensions and integration for the Hammer 3D level editor.
*   **`ShaderGraph/`:** The visual material editor.
*   **`MovieMaker/`:** The timeline-based cinematic animation tool.

## Troubleshooting

:::warning Core Modification Gotchas
- **Do not edit core files:** If you modify a file in `game/core/materials/`, your changes will be completely erased the next time you update the s&box engine. 
- **Overriding safely:** If you want to change how a core file looks, copy the file into your own addon (maintaining the exact same directory structure, e.g., `myaddon/materials/dev/...`). The engine mounts your addon *after* core, so your file will override the core file without destroying the original.
:::

## Performance
When your game mounts the `game/core` folder, the engine must evaluate its contents.
- **Shader Compilation:** Shaders in `game/core/shaders/` compile at runtime when first encountered by the player. Avoid overriding core shaders unnecessarily, as doing so will break the built-in shader cache and cause extreme stuttering for players when they join your game.
- **Asset Lookups:** Looking up materials in the `core` directory using `Material.Load` is fast due to virtual filesystem hashing, but you should still cache material references in your `Component.OnStart` method rather than calling `Material.Load` continuously inside the `OnUpdate` hot path.

- **Missing Core Materials:** If an asset in `game/core/` is somehow deleted or unmounted (which is rare), the engine will display a pink-and-black checkerboard fallback texture. 
- **Accidental Overrides:** If you name your own asset `materials/dev/dev_measuregeneric01b.vmat`, the engine will prioritize your addon's asset over the core one. This is powerful but can lead to unexpected edge cases if you don't realize you are overriding global development textures used by all maps.

## Sample Projects
- [**Sweeper Sample**](../../build-games/samples-templates/sweeper.md) - This sample relies on the standard Razor UI elements and shaders provided by the core engine. Review it to see how a complete project references core content efficiently.

## Related Pages
- [Mounting System](../../systems/mounting/index.md)
- [Mounting Internals](../../engine-internals/mounting.md)
