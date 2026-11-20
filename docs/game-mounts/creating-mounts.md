---
title: "Creating Mounts"
icon: "🛠️"
created: 2026-04-19
updated: 2026-11-20
---

# Creating Mounts

The mount system is extensible - anyone can write a mount to add support for a new game or engine, and contribute it to s&box via pull request to the [sbox-mounts](https://github.com/Facepunch/sbox-mounts/) repo. A mount detects a game's install directory via Steam, scans its files, and converts assets into s&box compatible assets, all at runtime.

## Overview

The mounting system is built on two key classes:

- **`BaseGameMount`** - detects the game and registers assets during mount
- **`ResourceLoader`** - loads and converts individual assets on demand

The mounts included in s&box live in the [sbox-mounts](https://github.com/Facepunch/sbox-mounts/) repo. To contribute support for a new game or engine, create a new addon project, write the mount, and submit a pull request.

## Creating a Game Mount

Extend `BaseGameMount` and implement the required properties and methods:

```csharp
using System.IO;
using System.Threading.Tasks;
using Sandbox.Mounting;

/// <summary>
/// Registers loose model files from a game's installation.
/// </summary>
public class MyGameMount : BaseGameMount
{
	/// <summary>
	/// Unique identifier used in mount paths.
	/// </summary>
	public override string Ident => "rust";

	/// <summary>
	/// Display name shown in the editor.
	/// </summary>
	public override string Title => "Rust";

	/// <summary>
	/// Absolute path to the game's install folder.
	/// </summary>
	public string InstallDirectory { get; private set; }

	protected override void Initialize( InitializeContext context )
	{
		// Check if the game is installed via Steam
		if ( !context.IsAppInstalled( 252490 ) )
		{
			return;
		}

		InstallDirectory = context.GetAppDirectory( 252490 );
		IsInstalled = Directory.Exists( InstallDirectory );
	}

	protected override Task Mount( MountContext context )
	{
		// Scan game files and register each resource
		foreach ( var fullPath in Directory.EnumerateFiles( InstallDirectory, "*.mymodel", SearchOption.AllDirectories ) )
		{
			var path = Path.GetRelativePath( InstallDirectory, fullPath ).Replace( '\\', '/' );
			context.Add( ResourceType.Model, path, new MyModelLoader( path ) );
		}

		IsMounted = true;
		return Task.CompletedTask;
	}
}
```

This example uses Rust's Steam app ID for installation detection; `.mymodel` and the loader below are placeholders for the format you implement.

### Ident and Title

`Ident` is a unique lowercase identifier for your mount (e.g. `"quake"`, `"hl1"`). It's used in `mount://` paths and project config. `Title` is the display name shown in the editor.

### Initialize

Called at startup to detect whether the game is installed. Use `context.IsAppInstalled(appId)` to check Steam, and `context.GetAppDirectory(appId)` to get its install directory. This returns a directory path, or `null` if it is unavailable. Use `System.IO` to read loose files, or the game's archive reader for packed assets. Set `IsInstalled = true` after confirming the directory exists.

### Mount

Called when the user enables the mount. Scan the game's files and register each resource with `context.Add()`. Set `IsMounted = true` when done.

## Creating Resource Loaders

Each resource type needs a `ResourceLoader` that converts the game's native format into something s&box understands. Resources are loaded lazily - `Load()` is only called when the asset is actually used.

```csharp
/// <summary>
/// Reads and converts one model file from the mounted game.
/// </summary>
public class MyModelLoader : ResourceLoader<MyGameMount>
{
    private readonly string filePath;

    /// <summary>
    /// Stores a file path relative to the game's install directory.
    /// </summary>
    public MyModelLoader( string path )
    {
        filePath = path;
    }

    protected override object Load()
    {
        // Read the file and convert it to an S&box model
        var fullPath = System.IO.Path.Combine( Host.InstallDirectory, filePath );
        var data = System.IO.File.ReadAllBytes( fullPath );
        return ConvertToModel( data );
    }
}
```

Use the generic `ResourceLoader<T>` to get typed access to your mount instance via the `Host` property. Registering the loader with `context.Add()` assigns its host. Implement `ConvertToModel` for your game's format; it is a placeholder in this example.

### Async Loading

For resources that take time to load, override `LoadAsync` instead:

```csharp
protected override async Task<object> LoadAsync()
{
    var fullPath = System.IO.Path.Combine( Host.InstallDirectory, filePath );
    var data = await System.IO.File.ReadAllBytesAsync( fullPath );
    return ConvertToModel( data );
}
```

## Resource Types

Register resources using the `ResourceType` enum:

| Type         | S&box Extension | Description                                  |
|--------------|-----------------|----------------------------------------------|
| `Model`      | `.vmdl`         | 3D models with meshes, skeletons, animations |
| `Texture`    | `.vtex`         | Textures and images                          |
| `Material`   | `.vmat`         | Materials referencing textures               |
| `Sound`      | `.vsnd`         | Audio files                                  |
| `Scene`      | `.scene`        | Scene files                                  |
| `PrefabFile` | `.prefab`       | Prefab files                                 |

## Resource Flags

You can set flags on resources to control their behavior:

```csharp
var loader = new MyModelLoader( file );
loader.Flags = ResourceFlags.DeveloperOnly; // Only visible in asset browser, not in-game
```

| Flag            | Description                                         |
|-----------------|-----------------------------------------------------|
| `DeveloperOnly` | Hidden from game, only visible in the asset browser |
| `NeverRagdoll`  | Don't treat this model as ragdollable               |
| `Effect`        | Non-solid, no physics                               |

## NPC and Weapon Prefabs

A mount can supply working NPCs and weapons, not just their models. Register them as `ResourceType.PrefabFile` and have the loader return a `PrefabFile` built with `Sandbox.Mounting.PrefabBuilder`.

Use `WithName( Path )` to name the prefab after the loader's mount path. Build the GameObjects and components inside `builder.Scope()`, then call `builder.Create()` after leaving the scope. `WithMetadata( "Title", "Guard" )` sets the prefab's display title; `Description` and `Group` work the same way.

You can build on the engine's [NPC](../gameplay/npcs/index.md) and [weapon](../gameplay/inventory-weapons/weapons.md) components, or use your own. A converted model still needs an animator that understands its animations. The engine's `CitizenNpcAnimator` is for citizen graphs, not arbitrary rigs.

Register a prefab only when the installation contains the assets it needs.
