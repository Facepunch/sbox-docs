---
title: "Assets/Resources"
icon: "📦"
created: 2024-12-17
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Resources/ResourceLibrary.cs
  - engine/Sandbox.Engine/Resources/GameResource.cs
---

# Assets & Resources

Assets (or Resources) are data files that define content for your game, such as models, materials, and custom configurations. They are loaded efficiently at runtime and can be hotloaded in the editor.

```csharp
// Load a custom asset from your project
var myClothing = ResourceLibrary.Get<Clothing>( "config/tshirt.clothing" );

if ( myClothing != null )
{
    Log.Info( $"Loaded clothing: {myClothing.Title}" );
}
```

Instead of hardcoding values or writing complex parsing logic, you create simple `GameResource` classes and let the engine handle serialization, the inspector UI, and hotloading.

## Operations

### Loading Resources

You can load resources at runtime using `ResourceLibrary`. These paths are relative to your project's root folder.

```csharp
// TryGet is safer if you aren't sure the asset exists
if ( ResourceLibrary.TryGet<Clothing>( "config/tshirt.clothing", out var clothing ) )
{
    // Use your resource
}
```

### Exposing to the Inspector

Instead of loading resources manually by path (which is brittle if files move), the best pattern is to expose a property so level designers can drag and drop the asset onto the component.

```csharp
public sealed class MyOutfitSpawner : Component
{
    // Shows an asset picker in the Editor!
    [Property] public Clothing OutfitToSpawn { get; set; }

    protected override void OnStart()
    {
        if ( OutfitToSpawn != null )
        {
            Log.Info( $"Spawning {OutfitToSpawn.Title}..." );
        }
    }
}
```

## Troubleshooting

:::danger "TryGet returns false / Resource is null"
Double-check your file paths. Remember that `ResourceLibrary` expects the path to be relative to the game's mount points, typically including the extension (e.g. `items/sword.item`). Also, ensure the resource was actually saved and compiled in the editor.
:::

## Topics

* [**Custom Assets**](custom-assets.md) — Create your own data types that integrate with the inspector.
* [**Cloud Assets**](cloud-assets.md) — Fetch assets remotely at runtime.
* [**Binary Serialization**](binary-serialization.md) — How the engine serializes data.
* [**GameResource Extensions**](gameresource-extensions.md) — Extend built-in resource types.

## Related Guides

* [**Save and Load Data**](../../how-to/save-and-load.md)
