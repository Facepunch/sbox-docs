---
title: "Spawn Point"
icon: "♿"
sources:
  - engine/Sandbox.Engine/Scene/Components/Game/SpawnPoint.cs
updated: 2026-04-25
created: 2026-04-27
---

# Spawn Point

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `SpawnPoint` component.

The `SpawnPoint` component is a marker used to dictate where players or entities should spawn when they enter the game.

## Quick Working Example

```csharp
using Sandbox;
using System.Linq;

public sealed class CustomSpawner : Component
{
	[Property] public GameObject PlayerPrefab { get; set; }

	public void SpawnPlayer()
	{
		// Find all active spawn points in the scene
		var spawnPoints = Scene.GetAllComponents<SpawnPoint>().ToList();

		if ( spawnPoints.Count == 0 )
		{
			Log.Warning( "No spawn points found in the scene!" );
			return;
		}

		// Pick a random spawn point
		var randomSpawn = Game.Random.FromList( spawnPoints );

		// Spawn the player at that position and rotation
		var player = PlayerPrefab.Clone( randomSpawn.WorldPosition, randomSpawn.WorldRotation );
		Log.Info( $"Spawned player at {randomSpawn.GameObject.Name}" );
	}
}
```

### Multiplayer Spawning

If you are using the built-in `NetworkHelper` component to manage your multiplayer game, it automatically searches for `SpawnPoint` components in the scene when a new client joins and places their player prefab at one of these locations.

### Team-based Spawning

You can extend the basic `SpawnPoint` pattern by placing them on GameObjects with specific Tags. For example, tagging a `SpawnPoint` GameObject as `"red_team"` or `"blue_team"` allows your spawner to filter them.

```csharp
public Transform GetRedTeamSpawn()
{
	var redSpawns = Scene.GetAllComponents<SpawnPoint>()
		.Where( sp => sp.GameObject.Tags.Has( "red_team" ) )
		.ToList();

	return Game.Random.FromList( redSpawns ).Transform.World;
}
```

## Configuration

| Property | Description |
|:---|:---|
| `Color` | Changes the color of the editor gizmo icon and model, making it easier to organize team spawns visually in the editor. Default: `#E3510D` (Orange). |

## Troubleshooting

:::warning "Players are spawning stuck in the floor!"
Make sure your `SpawnPoint` GameObject is placed slightly above the ground, and its rotation is facing the correct way (the blue arrow points in the forward direction).
:::

:::danger "NetworkHelper says no spawn points found!"
Ensure your `SpawnPoint` GameObjects are active/enabled in the scene, and that they haven't been accidentally destroyed or disabled before the `NetworkHelper` tries to use them.
:::

## Related Pages
- [NetworkHelper](../../../systems/networking-multiplayer/index.md)
- [Scenes](../../scenes/index.md)
