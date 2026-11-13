---
title: "Creating Mounts"
icon: "🛠️"
created: 2026-04-19
updated: 2026-11-13
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
public class MyGameMount : BaseGameMount
{
	public override string Ident => "rust";
	public override string Title => "Rust";

	/// <summary>
	/// Read-only filesystem of the game's install folder
	/// </summary>
	public BaseFileSystem FileSystem { get; private set; }

	protected override void Initialize( InitializeContext context )
	{
		// Check if the game is installed via Steam
		if ( !context.IsAppInstalled( 252490 ) )
			return;

		FileSystem = context.GetFileSystem( 252490 );
		IsInstalled = true;
	}

	protected override Task Mount( MountContext context )
	{
		// Scan game files and register each resource
		foreach ( var path in FileSystem.FindFile( "/", "*.mymodel", recursive: true ) )
		{
			context.Add( ResourceType.Model, path, new MyModelLoader( this, path ) );
		}

		IsMounted = true;
		return Task.CompletedTask;
	}
}
```

### Ident and Title

`Ident` is a unique lowercase identifier for your mount (e.g. `"quake"`, `"hl1"`). It's used in `mount://` paths and project config. `Title` is the display name shown in the editor.

### Initialize

Called at startup to detect whether the game is installed. Use `context.IsAppInstalled(appId)` to check Steam, and `context.GetFileSystem(appId)` to get a read-only filesystem of the game's install directory. Set `IsInstalled = true` if the game is found.

### Mount

Called when the user enables the mount. Scan the game's files and register each resource with `context.Add()`. Set `IsMounted = true` when done.

## Creating Resource Loaders

Each resource type needs a `ResourceLoader` that converts the game's native format into something s&box understands. Resources are loaded lazily - `Load()` is only called when the asset is actually used.

```csharp
public class MyModelLoader : ResourceLoader<MyGameMount>
{
    private string FilePath;

    public MyModelLoader( MyGameMount host, string path )
    {
        FilePath = path;
    }

    protected override object Load()
    {
        // Read the file and convert it to an S&box model
        var data = Host.FileSystem.ReadAllBytes( FilePath );
        return ConvertToModel( data );
    }
}
```

Use the generic `ResourceLoader<T>` to get typed access to your mount instance via the `Host` property.

### Async Loading

For resources that take time to load, override `LoadAsync` instead:

```csharp
protected override async Task<object> LoadAsync()
{
    var data = await Host.FileSystem.ReadAllBytesAsync( FilePath );
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
var loader = new MyModelLoader( this, file );
loader.Flags = ResourceFlags.DeveloperOnly; // Only visible in asset browser, not in-game
```

| Flag            | Description                                         |
|-----------------|-----------------------------------------------------|
| `DeveloperOnly` | Hidden from game, only visible in the asset browser |
| `NeverRagdoll`  | Don't treat this model as ragdollable               |
| `Effect`        | Non-solid, no physics                               |
