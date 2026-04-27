---
title: "Change Scenes"
icon: "🖼️"
sources:
  - engine/Sandbox.Engine/Game/Game/Game.Scene.cs
  - engine/Sandbox.Engine/Scene/Scene/Scene.LoadSave.cs
  - engine/Sandbox.Engine/Systems/SceneSystem/SceneLoadOptions.cs
created: 2026-04-27
updated: 2026-04-27
---

# Change Scenes

Learn how to load, switch, and transition between different scenes in your game, both locally and in multiplayer.

## Quick Working Example

You can load a new scene directly by its file path using `Game.ActiveScene.LoadFromFile()`. This is useful for single-player games or menus.

```csharp
public sealed class SceneChanger : Component
{
	protected override void OnUpdate()
	{
		// Press Space to load the next level
		if ( Input.Pressed( "Jump" ) )
		{
			Game.ActiveScene.LoadFromFile( "scenes/level_02.scene" );
		}
	}
}
```

### Multiplayer: Changing the Scene as the Host

When you want to move all players from a lobby to a game level, the Host must call `Game.ChangeScene()`. 

```csharp
public void StartMatch()
{
	// Only the Host is allowed to change the scene for everyone
	if ( Networking.IsHost )
	{
		var options = new SceneLoadOptions();
		options.SetScene( "scenes/arena_01.scene" );
		
		Game.ChangeScene( options );
	}
}
```

### Loading with a SceneFile Resource

Instead of hardcoding a string path like `"scenes/menu.scene"`, you can expose a `SceneFile` property to the editor. This ensures the scene is cooked into the build and prevents typos.

```csharp
public sealed class MenuButton : Component
{
	[Property] public SceneFile TargetScene { get; set; }

	public void OnClickPlay()
	{
		if ( TargetScene != null )
		{
			// Load the scene using the dragged-in resource
			Game.ActiveScene.Load( TargetScene );
		}
	}
}
```

When using `SceneLoadOptions` or the `Load()` method, several configurations are available:

| Property | Description |
|:---|:---|
| `SetScene( string path )` | Sets the target scene by its relative file path. |
| `SetScene( SceneFile sf )` | Sets the target scene using a `SceneFile` resource reference. |
| `IsAdditive` | If true, loads the new scene's objects *into* the current scene without destroying existing objects. Defaults to `false`. |
| `ShowLoadingScreen` | If true, displays the default loading screen during the transition. Defaults to `true`. |
| `DeleteEverything` | If true, forces the deletion of all objects in the current scene, ignoring any `DontDestroyOnLoad` flags. Defaults to `false`. |

## Troubleshooting

:::warning "Scene.LoadFromFile: Couldn't find scene"
If you get this warning, double-check your spelling and path. File paths are relative to your project root (e.g., `"scenes/my_scene.scene"`). It's generally safer to use a `[Property] SceneFile` instead of raw strings.
:::

:::danger "Clients are getting out of sync or crashing on scene load"
If you are building a multiplayer game, **never** call `Game.ActiveScene.Load()` to change the main map. You must use `Game.ChangeScene( options )` from the Host. Using the local load method in multiplayer will cause the client to destroy their local networked objects while the server still thinks they exist, leading to massive desyncs.
:::

## Related Pages
- [Scene System](../scene/index.md)
- [Networking Overview](../systems/networking-multiplayer/index.md)
