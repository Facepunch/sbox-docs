---
title: "Create an Object Spawner"
icon: "✨"
sources:
  - engine/Sandbox.Engine/Scene/GameObject/GameObject.Clone.cs
  - engine/Sandbox.Engine/Utility/Time.cs
  - engine/Sandbox.Engine/Systems/Threads/TaskSource.cs
created: 2026-04-27
updated: 2026-04-27
---

# Create an Object Spawner

How to spawn prefabs repeatedly on a timer to create enemy waves, item drops, or continuous effects.

## Quick Working Example

```csharp
using Sandbox;

public sealed class ObjectSpawner : Component
{
	[Property] public GameObject PrefabToSpawn { get; set; }
	[Property] public float SpawnInterval { get; set; } = 2.0f;
	
	private float _timeUntilNextSpawn;

	protected override void OnStart()
	{
		// Set the initial timer
		_timeUntilNextSpawn = SpawnInterval;
	}

	protected override void OnUpdate()
	{
		if ( PrefabToSpawn == null ) return;

		// Decrease our timer by the time passed since the last frame
		_timeUntilNextSpawn -= Time.Delta;

		// When the timer reaches zero or goes negative, spawn the object!
		if ( _timeUntilNextSpawn <= 0f )
		{
			Spawn();
			
			// Reset the timer for the next spawn
			_timeUntilNextSpawn = SpawnInterval;
		}
	}

	private void Spawn()
	{
		// Clone the prefab at this spawner's exact position and rotation
		var spawnedObject = PrefabToSpawn.Clone( GameObject.WorldPosition, GameObject.WorldRotation );
		
		// Optional: We can do things to the new object right after spawning
		spawnedObject.Name = "Spawned Enemy";
	}
}
```

### Spawning using Async Tasks

Instead of manually counting down `Time.Delta` inside `OnUpdate`, you can use the asynchronous `Task` system. This is often cleaner for logic that needs to wait.

```csharp
public sealed class AsyncTaskSpawner : Component
{
	[Property] public GameObject PrefabToSpawn { get; set; }
	[Property] public float SpawnInterval { get; set; } = 2.0f;
	[Property] public int MaxSpawns { get; set; } = 10;
	
	private int _spawnCount = 0;

	protected override async void OnStart()
	{
		// Loop until we reach the max count, or the spawner is destroyed
		while ( IsValid && _spawnCount < MaxSpawns )
		{
			// Wait for the specified interval
			await Task.DelaySeconds( SpawnInterval );

			// Check IsValid again because the component might have been destroyed while waiting
			if ( !IsValid ) return;

			if ( PrefabToSpawn != null )
			{
				PrefabToSpawn.Clone( GameObject.WorldTransform );
				_spawnCount++;
			}
		}
	}
}
```

### Spawning in an Area

Instead of spawning exactly at the spawner's center, you might want to spawn enemies randomly within a radius.

```csharp
[Property] public float SpawnRadius { get; set; } = 200f;

private void SpawnRandom()
{
	// Generate a random 2D circle position, ignoring the Z axis (height)
	var randomOffset = Vector3.Random.WithZ( 0 ) * SpawnRadius;
	
	var spawnPos = GameObject.WorldPosition + randomOffset;
	
	PrefabToSpawn.Clone( spawnPos, Rotation.Identity );
}
```

When using `Clone()`, there are multiple ways you can pass the location and hierarchy:

| Method | Description |
|---|---|
| `.Clone()` | Spawns exactly at world origin `(0,0,0)`. |
| `.Clone( Vector3 position )` | Spawns at the specified position. |
| `.Clone( Vector3 position, Rotation rotation )` | Spawns at the specified position and rotation. |
| `.Clone( Transform transform )` | Uses a combined Transform struct for position, rotation, and scale. |

## Troubleshooting

:::danger NullReferenceException when Spawning!
If you forget to assign your `PrefabToSpawn` property in the Editor Inspector, it will be `null`. Calling `.Clone()` on a null object crashes the script. Always wrap your spawning logic in `if ( PrefabToSpawn != null )`.
:::

:::warning My Async spawner keeps throwing errors after I delete it!
When you use `await Task.DelaySeconds()`, the component waits in the background. If the player destroys the spawner GameObject during that wait, the component is no longer valid, but the code after `await` will still try to run. **Always check `if (!IsValid) return;` after an `await` call.**
:::

## Related Pages
- [Clone GameObjects](clone-gameobjects.md)
- [Spawn a Prefab](spawn-prefab.md)
- [GameObject Reference](../scene/gameobject.md)
