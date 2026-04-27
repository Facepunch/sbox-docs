---
title: "File System"
icon: "💾"
created: 2023-11-20
updated: 2026-04-27
sources:
  - engine/Sandbox.Filesystem/BaseFileSystem.cs
  - engine/Sandbox.Engine/Systems/Filesystem/FileSystem.cs
---

# File System

The File System restricts standard .NET file access (`System.IO.File`) to prevent rogue access, providing isolated virtual filesystems for safe data management within specific game directories.

## Quick Working Example

```csharp
using Sandbox;

public sealed class PlayerDataManager : Component
{
	public class PlayerData
	{
		// Must be properties to serialize correctly!
		public int Level { get; set; } = 1;
		public int MaxHealth { get; set; } = 100;
	}

	public void SavePlayerData( PlayerData data )
	{
		// Writes to the game's specific data directory
		FileSystem.Data.WriteJson( "player.json", data );
		Log.Info( "Player data saved!" );
	}

	public PlayerData LoadPlayerData()
	{
		// Reads from the game's specific data directory, or returns default if missing
		var data = FileSystem.Data.ReadJsonOrDefault<PlayerData>( "player.json", new PlayerData() );
		Log.Info( $"Player level: {data.Level}" );
		return data;
	}
}
```

### Reading and Writing Raw Text

If you don't need complex objects, you can read and write simple strings.

```csharp
// Write text, creating the file if it doesn't exist
if ( !FileSystem.Data.FileExists( "log.txt" ) )
{
    FileSystem.Data.WriteAllText( "log.txt", "Initial log entry." );
}

// Read the entire text file back
string logContents = FileSystem.Data.ReadAllText( "log.txt" );
```

### Reading and Writing Bytes

You can also interact with raw byte arrays, useful for binary data or custom formats.

```csharp
// Read bytes asynchronously to avoid freezing the main thread
byte[] data = await FileSystem.Data.ReadAllBytesAsync( "custom.bin" );
```

### Finding Files

You can search for specific files using wildcard patterns.

```csharp
// Find all JSON files in the root data directory
var saveFiles = FileSystem.Data.FindFile( "/", "*.json" );

foreach( var file in saveFiles )
{
    Log.Info( $"Found save: {file}" );
}
```

## Configuration

There are three primary filesystems you will use, accessible as static properties on `FileSystem`.

| FileSystem | Description | Writable? |
|:---|:---|:---|
| `FileSystem.Data` | Per-game user data. Use this for save files. | Yes |
| `FileSystem.OrganizationData` | User data shared across every game in your organization. | Yes |
| `FileSystem.Mounted` | Aggregate read-only view of mounted content — the core engine, the current game, and any downloaded addons. | No |

On disk, those resolve under your Steam library:

| FileSystem | Real path |
|:---|:---|
| `FileSystem.Data` | `…\steamapps\common\sbox\data\<org>\<game>\` |
| `FileSystem.OrganizationData` | `…\steamapps\common\sbox\data\<org>\` |
| `FileSystem.Mounted` | composite (engine + game + dependencies) |

You shouldn't write code that depends on those literal paths — use the `FileSystem` API — but knowing where files end up helps when debugging missing saves.

## Troubleshooting

:::danger "Variables aren't saving to JSON!"
`WriteJson` and `ReadJson` will **only** work with [Properties](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/properties) by default. Regular fields will be silently ignored. Ensure the variables you want to save look like `public int Level { get; set; }` and not `public int Level;`.
:::

:::warning "Access Denied / System.IO Exceptions"
If you attempt to use standard .NET `System.IO` methods to read or write files directly, the game will throw a `System.Security.SecurityException` or `UnauthorizedAccessException`. Always use the `FileSystem` API.
:::

## Related Pages
- [Storage (UGC)](storage-ugc.md)
- [Save and Load Data](../../how-to/save-and-load.md)
