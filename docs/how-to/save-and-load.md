---
title: "Save and Load Data"
icon: "💾"
sources:
  - engine/Sandbox.Filesystem/BaseFileSystem.cs
created: 2026-04-27
updated: 2026-04-27
---

# Save and Load Data

Learn how to persist player progress, settings, and game state between sessions using the FileSystem.

## Quick Working Example

```csharp
using Sandbox;

public sealed class SaveManager : Component
{
	public class PlayerSaveData
	{
		// Only Properties (with getters/setters) are serialized by default
		public string PlayerName { get; set; } = "Player";
		public int Level { get; set; } = 1;
		public float Health { get; set; } = 100f;
	}

	public void SaveGame( PlayerSaveData data )
	{
		// Serialize to JSON and write to the local game data folder
		FileSystem.Data.WriteJson( "savegame.json", data );
		Log.Info( "Game Saved!" );
	}

	public PlayerSaveData LoadGame()
	{
		// Read the JSON file back into an object, or return a default object if it doesn't exist
		var data = FileSystem.Data.ReadJsonOrDefault<PlayerSaveData>( "savegame.json", new PlayerSaveData() );
		Log.Info( $"Game Loaded: Welcome back {data.PlayerName}, Level {data.Level}" );
		return data;
	}
}
```

### Checking if a Save Exists

Before attempting to load or show a "Continue" button, you can check if the save file exists.

```csharp
if ( FileSystem.Data.FileExists( "savegame.json" ) )
{
	// Enable the Continue button on the main menu
}
```

### Saving Raw Text or Bytes

If you are not saving structured class data, you can write raw strings or byte arrays directly.

```csharp
// Write plain text
FileSystem.Data.WriteAllText( "log.txt", "The player entered the dungeon." );

// Read plain text
string logText = FileSystem.Data.ReadAllText( "log.txt" );
```

### Creating Directories

You might want to organize saves into folders, such as `/saves/slot1/`.

```csharp
// Create the directory if it doesn't exist
FileSystem.Data.CreateDirectory( "saves/slot1" );

FileSystem.Data.WriteJson( "saves/slot1/data.json", myData );
```

## Configuration

The `FileSystem` class provides several different isolated file systems depending on what you need to do.

| FileSystem | Description | Writable? |
|:---|:---|:---|
| `FileSystem.Data` | The primary location for saving persistent game data and settings for your project. | Yes |
| `FileSystem.Mounted` | Read-only access to the assets bundled with your game or downloaded addons (e.g., loading a `.vmdl` or `.json` shipped with your game). | No |
| `FileSystem.OrganizationData` | A shared directory for all games created by your specific Facepunch Organization. Good for shared settings across your franchise. | Yes |

## Troubleshooting

:::warning "I'm getting a FileNotFoundException when loading!"
Always check if the file exists first, or use `FileSystem.Data.ReadJsonOrDefault<T>` that accepts a `returnOnError` parameter to gracefully handle new players who haven't saved yet.
:::

:::danger "My complex C# object won't serialize!"
`WriteJson` relies on `System.Text.Json`. Ensure the variables you want to save are **Properties** (e.g. `public int Score { get; set; }`). Regular fields (`public int Score;`) are ignored by default. Furthermore, if you try to save an entire `GameObject` or `Component` directly, it will fail or cause a stack overflow due to circular references. Always map your game state to a simple "Data Object" (POCO) class before saving.
:::

## Related Pages
- [File System](../systems/file-system/index.md)
