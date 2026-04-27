---
title: "Custom Assets"
icon: "🧊"
sources:
  - engine/Sandbox.Engine/Resources/GameResource.cs
  - engine/Sandbox.Engine/Resources/ResourceLoader.cs
created: 2026-04-27
updated: 2026-04-27
---

# Custom Assets

Define your own data structures as `GameResource` classes to create hotloadable custom assets with auto-generated Inspector UI.

## Quick Start
```csharp
using Sandbox;

// Define a custom asset type with a file extension of .clothing
[AssetType( Name = "Clothing Definition", Extension = "clothing", Category = "citizen" )]
public partial class Clothing : GameResource
{
    public string Title { get; set; }
    
    [ResourceType( "vmdl" )]
    public string ModelPath { get; set; }
    
    public int Cost { get; set; } = 100;
}
```

## Usage Patterns

### Creating New Asset Files
Once your `GameResource` class is compiled, right-click in the Asset Browser and go to the "New" menu. Your asset will be listed there (under the specified `Category`, or "Other").

### Loading a Specific Asset
You can load a known asset by its file path using `ResourceLibrary.Get<T>` or `ResourceLibrary.TryGet<T>`.

```csharp
// Returns null if the asset cannot be found
var myShirt = ResourceLibrary.Get<Clothing>( "config/tshirt.clothing" );

if ( myShirt != null )
{
    Log.Info( $"Loaded {myShirt.Title} costing {myShirt.Cost} coins." );
}

// Alternatively, use TryGet which is slightly safer
if ( ResourceLibrary.TryGet<Clothing>( "config/tshirt.clothing", out var clothing ) )
{
    Log.Info( $"Loaded {clothing.Title}" );
}
```

### Keeping Track of All Instances
If you want a static list of all assets of this type loaded, you can override `PostLoad`. `PostLoad` is called automatically by the engine after the resource is loaded into memory.

```csharp
public partial class Clothing : GameResource
{
	// Access these statically with Clothing.All
	public static IReadOnlyList<Clothing> All => _all;
	internal static List<Clothing> _all = new();

	protected override void PostLoad()
	{
		base.PostLoad();

        // Since you are constructing the list yourself, you could add your own logic here
        // to create lists for All, AllHats, AllShirts, AllShoes, ect.
		if ( !_all.Contains( this ) )
			_all.Add( this );
	}
}
```

## Configuration

| Attribute | Description |
|---|---|
| `[AssetType(Name, Extension, Category)]` | Marks a `GameResource` class to be discoverable by the Asset Browser. The `Extension` **must be all lowercase and <= 8 characters**. |
| `[ResourceType("ext")]` | When placed on a string property, it tells the Inspector to show an asset picker specifically filtering for that file extension. |
| `[Hide]` | Hides the property from the Inspector. |

You can also use standard [Property Attributes](../../editor/property-attributes.md) to control the Inspector's UI.

## Troubleshooting

:::warning
**Custom Asset failing to save or register?**
Ensure the `Extension` parameter in your `[AssetType]` attribute is exactly 8 characters or fewer, and completely lowercase.
:::

## Related Pages
* [GameResource Extensions](gameresource-extensions.md)
* [Assets/Resources Overview](index.md)
