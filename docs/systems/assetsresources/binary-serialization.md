---
title: "Binary Serialization"
icon: "🔟"
sources:
  - engine/Sandbox.Engine/Utility/BlobData.cs
created: 2026-04-27
updated: 2026-04-27
---

# Binary Serialization

Store large or complex data in GameResources as a companion binary blob instead of human-readable JSON to optimize load times and file size.

## Quick Working Example

Create a class that inherits from `BlobData` and implement the `Serialize` and `Deserialize` methods. Then, use it as a property in your `GameResource`.

```csharp
using Sandbox;
using System.Collections.Generic;

// The GameResource will serialize 'Positions' as binary data
[GameResource( "Path Data", "path", "Custom path points" )]
public partial class PathResource : GameResource
{
	public string Title { get; set; }
	public PathBlob Positions { get; set; } = new();
}

// Inherit from BlobData to enable binary serialization
public class PathBlob : BlobData
{
	public List<Vector3> Points { get; set; } = new();

	public override void Serialize( ref Writer writer )
	{
		// Write the count first so we know how many to read back
		writer.Stream.Write( Points.Count ); 
		
		foreach ( var point in Points )
		{
			writer.Stream.Write( point );
		}
	}

	public override void Deserialize( ref Reader reader )
	{
		var count = reader.Stream.Read<int>();

		for ( int i = 0; i < count; i++ )
		{
			Points.Add( reader.Stream.Read<Vector3>() );
		}
	}
}
```

### Versioning and Upgrades

Because binary data is strict, changing your structure (e.g., adding a new field to your blob) will break deserialization for older files. `BlobData` provides a `Version` property and an `Upgrade` method to handle this gracefully.

```csharp
public class MyVersionedBlob : BlobData
{
	// Bump this version when you change the Serialize layout
	public override int Version => 2;
	
	public int Health { get; set; }
	public int Mana { get; set; } // Added in version 2

	public override void Serialize( ref Writer writer )
	{
		writer.Stream.Write( Health );
		writer.Stream.Write( Mana );
	}

	public override void Deserialize( ref Reader reader )
	{
		Health = reader.Stream.Read<int>();
		Mana = reader.Stream.Read<int>();
	}

	public override void Upgrade( ref Reader reader, int fromVersion )
	{
		if ( fromVersion == 1 )
		{
			// Read version 1 format
			Health = reader.Stream.Read<int>();
			
			// Set default for the new field
			Mana = 100; 
		}
		else
		{
			// If it's up to date, just deserialize normally
			Deserialize( ref reader );
		}
	}
}
```

## Configuration

| Method/Property | Description |
|:---|:---|
| `Serialize( ref Writer writer )` | Override to define how your data is written to the byte stream. |
| `Deserialize( ref Reader reader )` | Override to define how your data is read back from the byte stream. Must match the exact order of `Serialize`. |
| `Version` | Override to provide the current version number of your binary format layout. Default is `1`. |
| `Upgrade( ref Reader reader, int fromVersion )` | Override to implement custom reading logic for older versions of your binary layout. |

## Troubleshooting

:::danger "My binary data is corrupted after changing the class!"
If you add or remove properties from your `Serialize` method, you **must** update your `Deserialize` method to match the exact same order. Furthermore, if you are loading existing data created with the old format, you must increase the `Version` property and handle the old format in the `Upgrade` method, otherwise the stream will read the bytes out of order.
:::

## Related Pages
* [Custom Assets](custom-assets.md)
* [Assets/Resources Overview](index.md)

