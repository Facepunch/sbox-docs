---
title: Sandbox Addon Template
sources:
  - game/templates/sandbox.addon
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox Addon Template

The **Sandbox Addon** template (`game/templates/sandbox.addon/`) is a specialized starting point designed specifically for creating extensions to the official `sandbox` gamemode.

## Quick Working Example

When creating a sandbox addon, you generally expose custom GameObjects or tools to the Sandbox spawn menu. While this template is mostly empty initially, you populate it with Prefabs.

```csharp
// Example of a Sandbox Entity script you might include in this addon
using Sandbox;

// This attribute tells the official Sandbox game to show this in the Spawn Menu
[Spawnable] 
public sealed class BouncyBall : Component
{
    protected override void OnStart()
    {
        if ( GameObject.Components.TryGet<Rigidbody>( out var rb ) )
        {
             // Make the ball bouncy
             // ...
        }
    }
}
```

1. **Spawnable Props & Entities:** You create Prefabs with rigidbodies, custom scripts, and visual effects, then tag the root C# component with the `[Spawnable]` attribute. When a player mounts your addon in the Sandbox gamemode, your item will automatically appear in their "Q" spawn menu.
2. **Weapons & Tools:** You can distribute custom weapons designed specifically to integrate with the Sandbox gamemode's inventory and player controller.
3. **Map Asset Packs:** Providing a folder full of `.vmdl` files for map creators to use.

- **Sandbox API Dependency:** If you are writing C# code that specifically hooks into the Sandbox gamemode's internal logic (like its Toolgun system or its specific Player controller), your addon project must add `facepunch.sandbox` as a package dependency in your Project Settings. Otherwise, your code will fail to compile.

## Troubleshooting

:::warning Common Gotchas
- **Addon doesn't show up in the Sandbox spawn menu:** Ensure your root C# component has the `[Spawnable]` attribute. Without it, the Sandbox UI doesn't know your Prefab is intended to be spawned by players.
- **Can't launch project:** Like all addons, you cannot hit "Play" on a Sandbox Addon directly. You must launch the main Sandbox gamemode and ensure your local addon is selected in the active addon list before starting the map.
:::

## Related Pages
- [Minimal Addon Template](addon-minimal.md)
- [Publishing Addons](../../publishing/publishing-addons.md)
