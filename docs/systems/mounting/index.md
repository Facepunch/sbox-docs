---
title: Game Mounts
icon: "📂"
sources:
  - engine/Sandbox.Engine/Game/Mount/Directory.cs
  - engine/Mounting/Sandbox.Mounting/MountedGame/BaseGameMount.cs
  - engine/Mounting/Sandbox.Mounting/MountedAsset/ResourceLoader.cs
updated: 2026-04-25
created: 2026-04-27
---

# Game Mounts

Game Mounts let players use assets from other games they have installed inside s&box. Models, textures, materials, and sounds are converted on the fly — no manual importing or file conversion. A player who owns Quake on Steam can drop Quake monsters into a Sandbox Spawn Menu game; a player who owns NS2 can use NS2 textures in their map.

> **Mounted assets are runtime-only.** You cannot publish or ship them with your game. Players need to own and install the original game on Steam to use those assets.

## How it works

Each mount is a plugin that knows how to find and convert one specific game's assets. When the mount is enabled:

1. It detects whether the source game is installed by querying Steam.
2. It scans for the game's assets and registers them.
3. When something requests `mount://<ident>/path/to/asset.vmdl`, the mount loads the original file and converts it to s&box format on demand.

Conversions are lazy — assets aren't all converted up front, only when first requested. Mounting an entire BSP map for the first time will hitch briefly while the geometry, textures, and entities are translated to Source 2 equivalents.

Mounts are toggled in the **Asset Browser** or used in the **Sandbox Spawn Menu**. The system is extensible — anyone can write a new mount and contribute it to s&box via pull request. See [Creating Mounts](creating-mounts.md).

## What's bundled

| Mount | Game | Source | Page |
|---|---|---|---|
| `goldsrc` | Half-Life 1, Counter-Strike 1.6, etc. | `engine/Mounting/Sandbox.Mounting.GoldSrc/` | [GoldSrc](goldsrc.md) |
| `quake` | Quake (1996) | `engine/Mounting/Sandbox.Mounting.Quake/` | [Quake](quake.md) |
| `ns2` | Natural Selection 2 | `engine/Mounting/Sandbox.Mounting.NS2/` | [NS2](ns2.md) |

## Querying mounts from code

Use `Sandbox.Mounting.Directory` to enumerate available mounts and check their state:

```csharp
using Sandbox.Mounting;

// Information about every registered mount, installed or not.
var mounts = Directory.GetAll();
foreach ( var info in mounts )
{
    Log.Info( $"{info.Title} ({info.Ident}): installed={info.IsInstalled}, mounted={info.IsMounted}" );
}

// Look up a specific mount.
var quake = Directory.Get( "quake" );
if ( quake is { IsInstalled: true, IsMounted: false } )
{
    await Directory.Mount( "quake" );
}
```

`Directory.GetAll()` returns `MountInfo[]` (a serializable view); `Directory.Get(name)` returns the live `BaseGameMount`. `Directory.Mount(name)` enables a mount asynchronously and returns the mount object once enabled, or `null` if the source isn't installed.

## Loading mounted assets

Mounted assets live at `mount://<ident>/<path>`. Pass that URL to the normal resource loaders:

```csharp
var monster = Model.Load( "mount://quake/maps/e1m1/monster_grunt.vmdl" );
var wall = Material.Load( "mount://goldsrc/textures/wall_brick.vmat" );
var clip = SoundEvent.Load( "mount://ns2/sounds/marines/footstep.sound" );
```

If the mount isn't enabled, the load returns the missing-asset placeholder. There's no special API needed beyond the URL prefix.

- **First load is slow.** Converting a legacy BSP into a modern scene takes seconds, not milliseconds. Cache references, don't reload per frame.
- **Visual mismatch.** Materials from 1998 don't translate cleanly to PBR. Expect classic assets to look matte and wrong-shiny without a manual material pass.
- **Missing game.** If `IsInstalled` is false, the mount can register at boot but `Mount()` won't actually load anything. Always provide a fallback if your code depends on a specific mount.

## See also

- [Creating Mounts](creating-mounts.md) — write a new mount and contribute it.
- [Mounting (Engine Concepts)](../../engine-concepts/mounting.md) — conceptual model.
- [Mounting Internals](../../engine-internals/mounting.md) — engine-side reference.
