---
title: Quake Mounting
icon: "🧠"
sources:
  - game/mount/quake/
updated: 2026-04-27
created: 2026-04-27
---

# Quake Mounting

The Quake mounter allows developers to use assets, maps, and textures from the original Quake directly inside s&box by dynamically translating them at runtime.

## Quick Start Example

You can check for the Quake mount and load assets from it if available:

```csharp
using Sandbox;
using Sandbox.Mounting;

public sealed class QuakeTest : Component
{
    protected override void OnStart()
    {
        // AppId 2310 corresponds to Quake
        var quakeMount = Sandbox.Mounting.Directory.GetAll().FirstOrDefault( x => x.AppId == 2310 );
        if ( quakeMount != null && quakeMount.IsInstalled )
        {
            Log.Info( "Quake assets are mounted and ready." );
            // You can load quake models or maps
        }
    }
}
```

## Common Patterns

1. **BSP Conversion:** Automatically converts classic Quake maps into navigable Source 2 geometry.
2. **Palette Mapping:** Converts the specific 256-color palette used by Quake into modern textures so they render correctly in a PBR environment.
3. **Audio Translation:** Handles any necessary sample rate conversions for Quake's legacy audio files.

## Visual Diagnostics

You can verify the Quake asset translation process visually by opening the Editor's Asset Browser. When the mount is active, Quake `.bsp` and `.mdl` files will be visible. Clicking them will force the C# parser to execute and render the translated output in the Inspector preview window, allowing you to instantly check for palette or vertex parsing errors.

## Troubleshooting

:::warning
- **Missing Installation:** As with all mounts, the player must have Quake installed on Steam.
- **Fullbright Textures:** If the palette conversion fails or is missing a specific color index, some textures may render as completely white or pink/black checkerboards.
:::

## Sample Validation
You can examine the open-source implementation within `game/mount/quake/` to understand how `Sandbox.Mounting` intercepts and transforms completely un-rigged vertex animation data into modern rendering submissions.

## Related Pages
- [External Game Mounting](index.md)
- [Mounting Internals](../../engine-internals/mounting.md)
