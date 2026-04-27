---
title: "Storage (UGC)"
icon: "📂"
sources:
  - engine/Sandbox.Engine/Systems/Filesystem/Storage/Storage.cs
  - engine/Sandbox.Engine/Systems/Filesystem/Storage/Storage.Entry.cs
  - engine/Sandbox.Engine/Systems/Filesystem/Storage/Storage.Query.cs
created: 2026-04-27
updated: 2026-04-27
---

# Storage (UGC)

The **Storage** system provides a unified API to manage User Generated Content (UGC) like save games and player creations, effortlessly bridging local file management and Steam Workshop integration.

## Quick Working Example

You can create a self-contained storage entry, save files into it, add metadata, and even publish it to the Steam Workshop.

```csharp
using Sandbox;

public class SaveManager : Component
{
	public void CreateSaveGame()
	{
		// 1. Create a new storage entry of type "save"
		var saveEntry = Storage.CreateEntry( "save" );

		// 2. Write your game data directly to its internal filesystem
		saveEntry.Files.WriteAllText( "player.json", "{ \"health\": 100, \"coins\": 50 }" );

		// 3. Set metadata for searching and UI display
		saveEntry.SetMeta( "playerName", "John Doe" );
		saveEntry.SetMeta( "level", 42 );

		Log.Info( $"Created save game with ID: {saveEntry.Id}" );
	}
	
	public void LoadSaves()
	{
		// 4. Retrieve all storage entries of type "save" from disk
		var allSaves = Storage.GetAll( "save" );

		foreach ( var save in allSaves )
		{
			var playerName = save.GetMeta<string>( "playerName" );
			Log.Info( $"Found save for player: {playerName}" );
		}
	}
}
```

### Working with Files
Each `Entry` exposes a `Files` property, which is a fully-featured `BaseFileSystem`. You can read, write, and create directories just like you would with the main game filesystem, but restricted to this entry's folder.

```csharp
var entry = Storage.CreateEntry( "screenshot" );

// Write files
entry.Files.WriteAllText( "notes.txt", "This was a cool moment!" );
entry.Files.WriteJson( "data.json", new { Score = 100 } );

// Read files
if ( entry.Files.FileExists( "data.json" ) )
{
    var data = entry.Files.ReadAllText( "data.json" );
}
```

### Thumbnails
You can attach an image to an entry, which is used natively by the UI or when uploading to Steam Workshop.

```csharp
// Assuming you have generated or captured a Bitmap
Bitmap myScreenshotBitmap = Screen.TakeScreenshot();

var entry = Storage.CreateEntry( "save" );
entry.SetThumbnail( myScreenshotBitmap );

// Access it later
Texture thumbnail = entry.Thumbnail;
```

### Publishing to Steam Workshop
Publishing an entry to the Steam Workshop opens a dialog for the user, allowing them to name the entry, add a description, and change visibility.

```csharp
var entry = Storage.CreateEntry( "dupe" );
// ... save data to entry.Files ...

// Publishes the entry to the workshop
entry.Publish( new WorkshopPublishOptions 
{ 
	Title = "My Cool Creation" 
} );
```

### Querying the Workshop
You can search the Steam Workshop for entries created by the community.

```csharp
public async Task FindAdventureSaves()
{
	var query = new Storage.Query
	{
		TagsRequired = { "save", "adventure" },
		SearchText = "epic quest",
		SortOrder = Storage.SortOrder.RankedByVote
	};

	var result = await query.Run();
	Log.Info( $"Found {result.TotalCount} items!" );
	
	foreach( var item in result.Items )
	{
	    Log.Info( $"{item.Title} by {item.Owner.Name}" );
	}
}
```

### Downloading Workshop Items
Once you query a `Storage.QueryItem`, you can download and convert it into a local `Storage.Entry`.

```csharp
Storage.Entry entry = await chosenItem.Install();

// Workshop entries are read-only!
if ( entry.Files.IsReadOnly )
{
	var data = entry.Files.ReadAllText( "player.json" );
	Log.Info( "Loaded community save file!" );
}
```

## Configuration

| Property/Method | Description |
|---|---|
| `Id` | A unique string identifier (GUID) for the entry. |
| `Type` | The category type string (e.g., `"save"` or `"dupe"`). Must be 1-16 letters long. |
| `Created` | The `DateTimeOffset` when the entry was created. |
| `Files` | The `BaseFileSystem` used for reading and writing files within the entry. |
| `Thumbnail` | The thumbnail image as a `Texture`. |
| `SetMeta<T>()` / `GetMeta<T>()` | Methods to read/write JSON-serializable metadata to the entry's internal `_meta.json` file. |
| `Delete()` | Deletes the entry and all its files from the local disk. |

## Troubleshooting

:::danger "System.ArgumentException: Invalid storage type"
The `type` argument passed to `Storage.CreateEntry( type )` must be 1 to 16 characters long and **only contain letters** (a-z, A-Z). You cannot use numbers, spaces, or symbols.
:::

:::warning "I can't write to a downloaded Workshop entry!"
When you call `await chosenItem.Install()`, the resulting `Storage.Entry` is stored in the user's Steam cache and is **read-only**. If you try to use `entry.Files.WriteAllText()`, it will fail or be ignored. If you need to modify downloaded content, you must copy the files into a new, locally created `Storage.Entry`.
:::

## Related Pages
- [File System Overview](index.md)
