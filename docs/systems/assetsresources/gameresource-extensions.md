---
title: "GameResource Extensions"
icon: "🔧"
sources:
  - engine/Sandbox.Engine/Resources/ResourceExtension.cs
created: 2026-04-27
updated: 2026-04-27
---

# GameResource Extensions

Resource Extensions allow you to append extra data fields to existing `GameResource` objects without modifying the original source code or asset files.

## Quick Start
```csharp
using Sandbox;

// 1. Define your extension using [GameResource]
[GameResource("Clothing Extension", "extcloth", "Extra Clothing Stuff", Category = "Citizen", Icon = "iron")]
public class ClothingExtension : ResourceExtension<Clothing, ClothingExtension>
{
    public string CustomCategory { get; set; } = "Other";
    public int CostToUnlock { get; set; } = 100;
}

// 2. Fetch the extension data for a specific asset anywhere in your game
var playerHat = ResourceLibrary.Get<Clothing>( "config/hat.clothing" );
var ext = ClothingExtension.FindForResourceOrDefault( playerHat );

if ( ext != null )
{
    Log.Info( $"The cost of the hat is {ext.CostToUnlock}" );
}
```

### Configuring the Extension in the Editor
1. Compile your `ResourceExtension` class.
2. In the Asset Browser, right-click -> **New** -> your extension type.
3. Set your custom values in the Inspector.
4. Open the **Extends** tab and select the existing assets you want this data attached to. You can also check "Default" to make this extension apply to any targeted asset that lacks its own specific extension.

### Retrieving Extension Data
You can look up extension data for any targeted asset at runtime using static helper methods automatically generated on your extension class.

```csharp
var playerHat = ResourceLibrary.Get<Clothing>( "config/hat.clothing" );

// Find specific extension, or fallback to the Default if none is found
var ext = ClothingExtension.FindForResourceOrDefault( playerHat );

// Find all extensions that are explicitly targeting the given asset
var allExts = ClothingExtension.FindAllForResource( playerHat );

// Get the extension explicitly marked as "Default"
var defaultExt = ClothingExtension.FindDefault();
```

## Configuration

Static lookup methods provided by inheriting from `ResourceExtension<TTarget, TSelf>`:

| Method | Returns | Description |
|---|---|---|
| `FindForResourceOrDefault( Resource r )` | `TSelf` | The extension specifically targeting the resource, or the default extension if not found. |
| `FindDefault()` | `TSelf` | The extension marked as the "Default" extension in the Inspector. |
| `FindAllForResource( Resource r )` | `IEnumerable<TSelf>` | All extensions targeting the specified resource. |

## Troubleshooting

:::warning
**Extension not found for an asset?**
Ensure you have assigned the target asset in the extension's "Extends" tab within the Inspector. Alternatively, make sure your extension is configured as the "Default".
:::

## Related Pages
* [Custom Assets](custom-assets.md)
* [Assets/Resources Overview](index.md)
