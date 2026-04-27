---
title: Natural Selection 2 Mounting
icon: "🧠"
sources:
  - game/mount/ns2/
updated: 2026-04-27
created: 2026-04-27
---

# Natural Selection 2 Mounting

The NS2 mounter translates assets from the Spark engine (used by Natural Selection 2) directly into s&box, allowing game developers to utilize its distinct sci-fi models, materials, and sounds.

## Quick Start Example

You can dynamically check for the presence of the NS2 mount before attempting to spawn its specific assets:

```csharp
using Sandbox;
using Sandbox.Mounting;

public sealed class NS2Test : Component
{
    protected override void OnStart()
    {
        // AppId 4920 corresponds to Natural Selection 2
        var ns2Mount = Sandbox.Mounting.Directory.GetAll().FirstOrDefault( x => x.AppId == 4920 );
        if ( ns2Mount != null && ns2Mount.IsInstalled )
        {
            Log.Info( "Natural Selection 2 assets are mounted and ready." );
            // E.g., load a skulk model or marine weapon
        }
    }
}
```

## Common Patterns

1. **Asset Interception:** File reads targeting the NS2 directory are intercepted and routed through the NS2-specific parser.
2. **Material Generation:** Spark engine material definitions often use different names for PBR channels (e.g., specular gloss vs roughness). The mounter automatically maps these to the standard Source 2 complex shader parameters.
3. **Model Parsing:** Handles the conversion of Spark's bone weights and mesh data into a format compatible with `SkinnedModelRenderer`.

## Visual Diagnostics
Just like GoldSrc mounting, you can visually verify the NS2 asset translation process by opening the Editor's Asset Browser. Navigating to the mounted NS2 folders and clicking on Spark engine `.model` files will force the C# parser to execute, rendering the resulting translated mesh in the Inspector preview window.

## Troubleshooting

:::warning
- **Missing Game Installation:** Players must own and have installed Natural Selection 2 on Steam for these assets to be available. If they don't, your `Model.Load` calls will fail silently or return error models.
- **Lighting Artifacts:** Because Spark engine materials are mapped dynamically, some very specific shader effects will fall back to standard PBR, losing their original visual complexity.
:::

## Sample Validation
You can examine the open-source implementation within `game/mount/ns2/` to see exactly how the `Sandbox.Mounting` API is utilized to register file interceptors and binary parsers for an engine completely distinct from the Source lineage.

## Related Pages
- [External Game Mounting](index.md)
- [Mounting Internals](../../engine-internals/mounting.md)
