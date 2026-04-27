---
title: "Scenes"
icon: "🌍"
created: 2023-10-26
updated: 2025-06-15
sources:
  - engine/Sandbox.Engine/Scene/Scene/Scene.cs
  - engine/Sandbox.Engine/Scene/Scene/Scene.LoadSave.cs
  - engine/Sandbox.Engine/Scene/Scene/Scene.ObjectIndex.cs
  - engine/Sandbox.Engine/Scene/Scene/GameObjectDirectory.cs
---

# Scenes

A Scene is the container for all GameObjects, Components, and systems that make up a level or environment in your game.

## Quick Working Example

```csharp
using Sandbox;

public class LevelManager : Component
{
	[Property] public SceneFile NextLevel { get; set; }

	public void GoToNextLevel()
	{
		if ( NextLevel == null ) return;
		
		// Replaces the current scene with the specified scene
		Scene.Load( NextLevel );
	}
}
```

### Getting the Current Scene

If you are writing code inside a `Component` or a `Panel`, you can access the scene the object belongs to via the `Scene` property.

```csharp
var myScene = this.Scene;
```

Alternatively, you can get the globally active scene:

```csharp
var currentScene = Game.ActiveScene;
```

### Loading a New Scene

You can load a scene using its `SceneFile` asset, or by a file path.

```csharp
// Load using a file path
Scene.LoadFromFile( "scenes/mainmenu.scene" );
```

To load a scene without unloading the current one, use `SceneLoadOptions`. This is useful for open-world streaming or loading a common UI scene on top of a game scene.

```csharp
var load = new SceneLoadOptions();
load.SetScene( myNewScene );
load.IsAdditive = true;

Scene.Load( load );
```

### Finding Objects (Directory)

Every scene maintains a `GameObjectDirectory`. This index assigns a unique GUID to every GameObject, allowing you to instantly look up any object in the scene.

```csharp
// Find an object by its unique ID
var targetObj = Scene.Directory.FindByGuid( myGuid );

// Loop over every GameObject in the scene
foreach ( var obj in Scene.Directory.AllGameObjects )
{
	Log.Info( $"Found: {obj.Name}" );
}
```

### Finding Components Fast

The scene automatically indexes all active components by their type. You can use `Scene.GetAll<T>()` to instantly retrieve all components of a specific type without manually searching the hierarchy.

```csharp
// Tint all models in the scene a random color
foreach ( var model in Scene.GetAll<ModelRenderer>() )
{
	model.Tint = Color.Random;
}

// Quickly grab a singleton component (e.g., PlayerController)
var player = Scene.Get<PlayerController>();
```

## Troubleshooting

:::danger NullReferenceException when calling Scene.Load
If `Game.ActiveScene` or your `SceneFile` property is null, you will get an error when trying to load. Always ensure your SceneFile property is assigned in the Inspector.
:::

:::warning Scene.GetAll returns empty?
`Scene.GetAll<T>()` only returns components that are currently active and initialized in the scene. If a GameObject or the Component itself is disabled, it will not appear in this list.
:::

## Related Pages

* [GameObject](../gameobject.md)
* [Components](../components/index.md)
* [Tracing](tracing.md)
