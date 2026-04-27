---
title: "Ready-to-use Assets"
icon: "✨"
created: 2025-04-29
updated: 2026-04-25
sources:
  - game/addons/citizen
  - game/addons/base
---

# Ready-to-use Assets

s&box includes a library of high-quality, ready-to-use assets—such as characters, weapons, and environmental props—that you can immediately drop into your project to start prototyping or building your game.

## Quick Working Example

You can spawn built-in assets by referencing their compiled `.vmdl` path. These paths are available to any project that depends on the standard `base` addon.

```csharp
using Sandbox;

public sealed class PropSpawner : Component
{
    protected override void OnStart()
    {
        // Spawns a standard wooden crate from the base addon
        var propObj = new GameObject(true, "Crate");
        var modelRenderer = propObj.Components.Create<ModelRenderer>();
        modelRenderer.Model = Model.Load( "models/sbox_props/wooden_crate/wooden_crate.vmdl" );
    }
}
```

1. **Prototyping with Citizen Characters:**
   Instead of using simple capsules for players or enemies, you can immediately use the fully rigged and animated [Citizen Character](citizen-characters.md). This saves weeks of technical art setup and allows you to test movement and combat mechanics on day one.

2. **First Person Weapons:**
   The base engine includes several fully animated [First Person Weapons](first-person-weapons.md) (like a pistol and a shotgun). You can attach these to your player's camera to instantly have access to firing, reloading, and idle animations.

3. **Level Blockouts with Dev Textures:**
   The engine provides a complete suite of developer textures (`materials/dev/`) that include grid measurements and color-coded materials. Use these when greyboxing your levels to ensure consistent scale before applying final art.

## Navigation Hub

* [**Citizen Characters**](citizen-characters.md) - The standard, fully rigged player models.
* [**First Person Weapons**](first-person-weapons.md) - High quality weapon viewmodels with animations.

## Troubleshooting

:::warning Assets Appearing as ERROR
If a ready-to-use asset appears as a giant red `ERROR` model, it means the engine cannot find it. Ensure that you have not typed the file path incorrectly, and verify that your project's `.sbproj` file includes `facepunch.base` (or `facepunch.citizen` for characters) in its dependencies.
:::

:::warning Pink and Black Checkerboards
If the model loads but its texture is a pink and black checkerboard, the material path is missing or the material failed to compile. This rarely happens with built-in assets, but if it does, try restarting the editor or verifying your s&box installation via Steam.
:::

Because ready-to-use assets are part of the core engine addons, they are often already cached in memory by the engine. Reusing these assets across your game (like using the standard Citizen model for 50 NPCs) is highly efficient due to GPU instancing.

## Related Pages
* [Assets Overview](../index.md)
