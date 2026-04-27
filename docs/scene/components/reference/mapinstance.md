---
title: "Map Instance"
icon: "🌍"
sources:
  - engine/Sandbox.Engine/Scene/Components/Map/MapInstance.cs
updated: 2026-04-25
created: 2026-04-27
---

# Map Instance

> **New to maps?** This is the reference for the `MapInstance` component. For the bigger picture see **[Maps](../../maps/index.md)** and **[Scene Mapping](../../scene-mapping/index.md)**.

The `MapInstance` component allows you to load an external map (a `.scene` authored with [Scene Mapping](../../scene-mapping/index.md), a Hammer `.vmap`, a compiled `.vpk`, or a published package ident) dynamically into your Scene.

## Quick Working Example

```csharp
using Sandbox;

public sealed class LevelLoader : Component
{
	[Property] public MapInstance MapSystem { get; set; }

	public void LoadLevel()
	{
		// Tell the MapInstance to load a specific map by its name
		MapSystem.MapName = "facepunch.construct";
		
		// You can also subscribe to events when it finishes loading
		MapSystem.OnMapLoaded += () =>
		{
			Log.Info( "Map successfully loaded!" );
		};
	}
}
```

### Waiting for a Map to Load

Because maps are often downloaded from the cloud and parsed asynchronously, loading takes time. You can use the `OnMapLoaded` event, or await the component's load during `OnLoad`.

```csharp
protected override async Task OnLoad( LoadingContext context )
{
	// If the MapInstance is already on the GameObject and enabled,
	// it will automatically start loading its map during OnLoad.
	Log.Info("Waiting for map to load...");
}
```

### Accessing Map Data

Once a map is loaded, you can get its physical bounds via the `Bounds` property. This is useful for placing boundaries or calculating camera constraints.

```csharp
public void PrintMapBounds()
{
	if ( MapSystem.IsLoaded )
	{
		BBox mapBounds = MapSystem.Bounds;
		Log.Info( $"The map is {mapBounds.Size} units large." );
	}
}
```

## Configuration

| Property | Description |
|:---|:---|
| `MapName` | The asset path or package ident (e.g., `"facepunch.construct"`) of the map to load. |
| `UseMapFromLaunch` | If true, it overrides `MapName` with whatever map was specified in the game's launch arguments. |
| `EnableCollision` | Whether the static map geometry should have physics collision enabled. Default is true. |
| `OnMapLoaded` | An Action event invoked when the map finishes loading successfully. |
| `OnMapUnloaded` | An Action event invoked when the map is unloaded. |

## Troubleshooting

:::warning "My networked props are missing when playing on a client!"
If you are spawning networked props (like physically simulated barrels) via a `MapInstance`, you must remember that only the **Host** or **Server** should spawn networked objects. The `MapInstance` automatically handles this: it will *not* spawn networked map objects if the game is running as a Client, expecting the Server to synchronize them instead.
:::

:::danger "Objects falling through the floor"
If your custom physics objects are falling through the map floor, ensure `EnableCollision` is set to `true` on the `MapInstance` component, and that the map itself was compiled with physics.
:::

## Related Pages
- [Maps overview](../../maps/index.md)
- [Loading Maps](../../maps/loading-maps.md)
- [Scene Mapping](../../scene-mapping/index.md) — the recommended path for new maps
- [Scenes](../../scenes/index.md)
