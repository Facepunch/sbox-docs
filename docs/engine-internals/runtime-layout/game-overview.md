---
title: Game Directory Overview
icon: "📁"
sources:
  - game
updated: 2026-04-25
created: 2026-04-27
---

# Game Directory Overview

The `game/` directory is the runtime environment of s&box. While `engine/` contains the source code to build the engine itself, `game/` contains everything needed to actually *run* games, including compiled binaries, core assets, and user-generated addons.

## Quick Start Example

You interact with the `game/` directory primarily through the s&box Editor, creating new projects inside it.

```csharp
using Sandbox;

public sealed class GamePathExample : Component
{
    protected override void OnStart()
    {
        // The engine relies heavily on a virtual filesystem mapping these files together at runtime.
        if ( FileSystem.Mounted.FileExists( "models/dev/box.vmdl" ) )
        {
            Log.Info( "The core resources have been mounted properly." );
        }
    }
}
```

## Common Patterns

1. **Working Directory:** When you run `sbox.exe`, its working directory is typically the `game/` folder. All relative file operations performed by the engine resolve relative to this root.
2. **Project Creation:** When you create a new game or addon in the editor, it is placed inside `game/addons/`. 
3. **Asset Mounting:** The engine mounts `game/core/` globally, and then dynamically mounts specific folders inside `game/addons/` based on the active project.

## Directory Layout

*   **[`addons/`](../../addons/index.md):** The primary location for all user-created content. Every project you make (games, libraries, UI elements) is an addon stored here.
*   **[`bin/`](game-core.md):** (Out of Scope for docs) Contains the compiled native Source 2 `.dll` and `.so` binaries required to run the engine.
*   **[`core/`](game-core.md):** Contains the fundamental assets (shaders, basic materials, editor tools) that all other addons rely on.
*   **[`editor/`](game-core.md):** Contains UI and logic specific to the s&box development environment (e.g., ShaderGraph, ActionGraph).
*   **[`mount/`](mount.md):** Contains the tools and configurations for mounting legacy game assets (like GoldSrc or Quake).
*   **[`samples/`](../../build-games/samples-templates/index.md):** Contains fully working sample games (like Sweeper) provided by Facepunch to study.
*   **[`templates/`](../../build-games/samples-templates/index.md):** Contains the bare-bones starting projects used when you click "New Project" in the editor.

## Troubleshooting

:::warning Directory Gotchas
- **Don't modify `core/`:** While you can edit files in `game/core/`, these will be overwritten the next time the engine updates. If you want to modify a core asset, copy it into your own addon and modify it there. The virtual filesystem will prefer your addon's file.
- **Accidental Deletions:** If you accidentally delete `game/bin/` or `game/core/`, the engine will not launch. You must run `Bootstrap.bat` (if building from source) or verify your files in Steam to restore them.
:::

## Related Pages
- [Runtime Layout & Architecture](index.md)
