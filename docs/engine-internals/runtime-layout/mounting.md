---
title: Mounting System
sources:
  - engine/Mounting
  - game/mount
updated: 2026-04-25
created: 2026-04-27
---

# Mounting System

> **See also:** [Mounting Internals](../mounting.md) — engine-developer deep dive (interceptors, `MountHost`, providers). [Mounting (Engine Concepts)](../../engine-concepts/mounting.md) — high-level conceptual overview. [External Game Mounting (Systems)](../../systems/mounting/index.md) — game-developer perspective on mounting GoldSrc/NS2/Quake content.

In s&box, "mounting" is the process of attaching a physical directory on your hard drive to the engine's virtual filesystem at runtime. This is how the engine knows where to find the models, materials, sounds, and scripts that make up a game or addon.

## How Mounting Works

When you play a game or load an addon, the engine doesn't just read files arbitrarily from your disk. Instead, it creates a virtual filesystem. 

For example, when you use a path like `models/my_prop.vmdl` in your code, the engine looks through all currently **mounted** directories to find that file.

### Addon Mounting

When an addon is loaded, its root folder (e.g., `game/addons/my_addon/`) is mounted. This allows the engine to access all the assets within that addon.

If a game relies on multiple addons (like the `base` addon and the `citizen` addon), all of them are mounted simultaneously. 

### Resolution Order

If two mounted addons contain a file with the exact same path (e.g., `materials/default.vmat`), the engine resolves this based on mounting priority. Typically, the game project itself has the highest priority, followed by its dependencies, and finally the `game/core` directory.

## Managing Mounts

The engine handles most mounting automatically based on your project's dependencies and configuration.

The `game/mount/` directory is used internally by the engine to manage and resolve these virtual filesystem paths at runtime.

### Example: Accessing Files

When writing code that accesses the filesystem (e.g., using `FileSystem.Mounted`), you are interacting with this virtual, mounted filesystem, not directly with your OS's hard drive paths.

```csharp
// Reads a file from the mounted filesystem
// This will search through the core, base, and any active addon folders
string content = FileSystem.Mounted.ReadAllText( "config/settings.json" );
```

Understanding the mounting system is critical for resolving "file not found" errors and structuring your addons correctly so their assets are visible to the engine.
