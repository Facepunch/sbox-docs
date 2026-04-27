---
title: GoldSrc Mounting
icon: "🧠"
sources:
  - game/mount/goldsrc/
updated: 2026-04-27
created: 2026-04-27
---

# GoldSrc Mounting

The GoldSrc mounter enables game developers to use assets from Half-Life 1, Counter-Strike 1.6, and other classic GoldSrc games directly within s&box by dynamically translating legacy formats into modern Source 2 representations at runtime.

## Quick Start Example

You can check for the presence of a GoldSrc mount (such as Half-Life, AppId 70) and react accordingly in your game logic:

```csharp
using Sandbox;
using Sandbox.Mounting;

public sealed class GoldSrcTest : Component
{
    protected override void OnStart()
    {
        // AppId 70 corresponds to Half-Life 1
        var goldSrcMount = Sandbox.Mounting.Directory.GetAll().FirstOrDefault( x => x.AppId == 70 );
        if ( goldSrcMount != null && goldSrcMount.IsInstalled )
        {
            Log.Info( "Half-Life 1 assets are mounted and ready." );
            // You can now load "models/scientist.mdl" and it will work!
        }
    }
}
```

## Common Patterns

1. **WAD Parsing:** Textures in GoldSrc are packed in `.wad` files. The mounter extracts these bitmaps and generates `.vmat` (materials) with correct PBR properties dynamically.
2. **MDL Translation:** Parses GoldSrc `.mdl` files to reconstruct a modern Source 2 skeleton, allowing legacy models to use standard `SkinnedModelRenderer` components.
3. **Map Parsing:** `.bsp` files are read and their brush geometry is converted to standard meshes, meaning you can load a classic Half-Life map as a scene.

## Visual Diagnostics
If you are developing or testing a GoldSrc mount, you can use the Editor's Asset Browser. When the mount is active, legacy folders will appear just like native assets. Clicking a `.bsp` or `.mdl` file will invoke the mounting layer and render a preview in the Inspector, allowing you to instantly visually verify that the C# parser correctly translated the vertices and textures.

## Troubleshooting

:::warning
- **"File not found: models/scientist.mdl":** This means the player does not have Half-Life 1 installed on Steam, or the mount was not properly initialized. Always check `IsInstalled` before attempting to load legacy assets, and provide fallback assets if necessary.
- **Weird Material Colors:** GoldSrc relies heavily on colormaps for team colors (like in Half-Life Deathmatch). The dynamic material generator attempts to emulate this, but some specific color values might look overly bright in a PBR environment.
:::

## Sample Projects
If you wish to see how complex mounting logic is managed, you can browse the open-source implementation in `game/mount/goldsrc/`.

## Related Pages
- [Mounting Internals](../../engine-internals/mounting.md)
- [Quake Mounting](quake.md)
