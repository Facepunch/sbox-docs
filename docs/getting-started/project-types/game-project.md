---
title: "Game Project"
icon: "🕹️"
sources:
  - engine/Sandbox.Engine/Systems/Project/Project/Project.cs
created: 2026-04-27
updated: 2026-04-27
---

# Game Project

A Game Project is a standalone experience containing everything needed to play a game, including scenes, code, maps, and assets.

## Quick Working Example

```csharp
using Sandbox;

public sealed class MyGameManager : Component, Component.ISceneStartup
{
	// This method is called automatically when the scene starts, 
	// before regular components run OnStart.
	void ISceneStartup.OnHostInitialize()
	{
		Log.Info("Game Project starting up!");
		
		// Load a default HUD or environment scene additively
		var slo = new SceneLoadOptions();
		slo.IsAdditive = true;
		slo.SetScene( "scenes/ui.scene" );
		Game.ActiveScene.Load( slo );
	}
}
```

### The Startup Scene

A Game Project generally defines a **Startup Scene** in its Project Settings. When a player launches your game, this scene is loaded first. 

*   **Menu First:** Typically, you set the startup scene to your main menu scene. The menu scene handles player lobbies, settings, and then uses `Game.ChangeScene()` to load the actual gameplay level.
*   **Direct to Game:** If your game is simple, you can bypass a menu entirely and set your primary gameplay scene as the startup scene.

### Map Bootstrapping

If your game allows players to load legacy BSP maps (like from Source 1 or early Source 2 games) directly from the main menu, the engine will load that map *instead* of a standard `.scene` file. 

Because maps don't natively contain your game's custom C# components (like UI, player spawners, or game managers), you need a way to inject them. You can use a `GameObjectSystem` or `ISceneStartup` (as shown in the Quick Example) to dynamically spawn your required game managers and UI into the scene as soon as the map loads.

## Configuration

You can configure Game Project settings in the Editor by going to **Edit -> Project Settings**.

| Setting | Description |
|---|---|
| `Startup Scene` | The `.scene` file that is loaded immediately when the game launches. |
| `Organization Ident` | The organization ID this game belongs to (e.g., `facepunch`). |
| `Package Ident` | The unique name of your game (e.g., `my_cool_game`). |

## Performance

When starting up a game project, large monolithic Startup Scenes can cause noticeable freezing for the player. It is often a better practice to have an incredibly lightweight loading scene (with a simple UI panel) as your `Startup Scene`, which then asynchronously loads the heavy gameplay scene in the background.

## Troubleshooting

:::danger "My game starts with an empty void!"
Ensure you have set a valid **Startup Scene** in your Project Settings. If no scene is specified, the engine will load an empty default scene.
:::

:::warning "Missing Assets on Publish"
If your Game Project uses assets from an Addon Project, ensure that Addon is explicitly listed as a dependency in your `.sbproj` file, otherwise players downloading your game will see pink checkerboards.
:::

## Related Pages
* [Project Types Index](index.md)
* [Addon Project](addon-project.md)
* [Exporting Standalone](../../publishing/exporting-standalone.md)
