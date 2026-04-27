---
title: "Assets"
icon: "🎎"
sources:
  - engine/Sandbox.Engine/Resources/ResourceLibrary.cs
created: 2026-04-27
updated: 2026-04-27
---

# Assets

Assets are the raw materials—models, textures, sounds, and custom resources—that make up your game's content.

## Quick Start
```csharp
using Sandbox;

// Loading an asset at runtime safely
if ( ResourceLibrary.TryGet<Model>( "models/dev/box.vmdl", out var myModel ) )
{
    // Model successfully loaded
    Log.Info($"Loaded {myModel.Name}");
}
```

## Common Patterns

1. **Inspector Assignment (Preferred):** Expose an asset type (like `Model` or `Texture`) as a `[Property]` in your Component, and assign it via the Editor GUI. This avoids hardcoding string paths and prevents typos.
2. **Runtime Loading:** Use `ResourceLibrary.Get<T>("path/to/asset.vmdl")` or `TryGet<T>` when you need to dynamically load an asset based on logic at runtime.
3. **Custom GameResources:** You can create your own custom asset types by creating a C# class that inherits from `GameResource`. This generates a `.asset` file that can be edited with an auto-generated inspector.
4. **Cloud Assets:** Assets can be loaded dynamically from the asset.party cloud using `Cloud.Asset(...)`, allowing you to fetch content at runtime without including it in your initial project size.

## Navigation Hub
* [**Ready to Use Assets**](ready-to-use-assets/index.md) - Explore the pre-made assets included in the engine.
* [**Clothing**](clothing/index.md) - Create custom clothes that are compatible with citizen characters.
* [**Custom Assets**](custom-assets.md) - Learn how to build data-driven GameResources.

## Troubleshooting
:::warning Common Gotchas
- **"ResourceLibrary.Get returns null!"**
  Ensure the asset path is exactly correct, including the file extension (e.g. `box.vmdl`). Ensure the asset actually exists in your project or an enabled Addon. Paths must be relative to the addon root.
- **"Asset doesn't update in-game after I changed the raw file."**
  The engine might not have recompiled it. Right click the asset in the Editor's Asset Browser and select "Recompile", or ensure the asset compilation tool isn't throwing errors in the console.
- **"Hardcoded paths break when sharing projects."**
  Always use relative paths starting from the addon root, rather than absolute system paths (e.g., `C:/...`).
:::

## Performance
- **Synchronous Loading:** Calling `ResourceLibrary.Get<T>` on a large asset (like a complex 3D model) during gameplay will cause a hitch on the main thread. It is highly recommended to use asynchronous loading (`ResourceLibrary.LoadAsync<T>`) for heavy assets needed during runtime.

## Related Pages
* [Editor](../editor/index.md)
* [Mounting System](../engine-internals/runtime-layout/mounting.md)
