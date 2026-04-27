---
title: External Game Mounting
icon: "📂"
sources:
  - engine/Mounting
updated: 2026-04-25
created: 2026-04-27
---

# External Game Mounting

> **See also:** [Mounting Internals](../engine-internals/mounting.md) — engine-developer deep dive (interceptors, `MountHost` lifecycle, mount-resolution algorithm). [External Game Mounting (Systems)](../systems/mounting/index.md) — game-developer perspective with GoldSrc/NS2/Quake specifics. [Mounting (Runtime Layout)](../engine-internals/runtime-layout/mounting.md) — VFS resolution-order summary.

The engine's `Mounting` system provides native support for loading and translating assets from older game engines (like GoldSrc, Quake, and Natural Selection 2) directly into s&box at runtime.

## Quick Start Example

You can mount legacy games via the s&box editor, or query mounted games through `Sandbox.Mounting.Directory`:

```csharp
using Sandbox;
using Sandbox.Mounting;

public sealed class MountExample : Component
{
    protected override void OnStart()
    {
        foreach ( var info in Directory.GetAll() )
        {
            Log.Info( $"{info.Title} ({info.Ident}): " +
                $"installed={info.IsInstalled}, mounted={info.IsMounted}" );
        }
    }
}
```

`Directory.GetAll()` returns `MountInfo[]`. To take action on a specific mount, use `Directory.Get(name)` or `Directory.Mount(name)`.

## Common Patterns

1. **Format Translation:** The core architecture is split into distinct assemblies for different engines (e.g., `Sandbox.Mounting.GoldSrc`, `Sandbox.Mounting.Quake`). Each assembly knows how to parse that specific engine's binary formats (like `.bsp` for maps, `.mdl` for models, `.wad` for textures).
2. **Virtual Filesystem Integration:** Mounted legacy games are injected into the standard virtual filesystem. Once mounted, you load a Quake sound using the exact same `Sound.Load()` API you use for native s&box sounds.

## Using Mounts

To mount a legacy game:
1. Open the s&box Editor.
2. Go to **File -> Mount Game...**
3. Select the directory of the legacy game you wish to mount.
4. The assets will now be available in your Asset Browser.

## Supported Engines

Currently, the engine provides built-in parsers for:
- **GoldSrc:** (Half-Life 1, Counter-Strike 1.6)
- **Quake:** (Quake 1)
- **NS2:** (Natural Selection 2)

## Related Pages
- [Runtime Layout](../engine-internals/runtime-layout/index.md)
