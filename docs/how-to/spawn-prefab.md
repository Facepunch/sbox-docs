---
title: "Spawn a Prefab"
icon: "🍳"
sources:
  - engine/Sandbox.Engine/Scene/GameObject/GameObject.Clone.cs
  - engine/Sandbox.Engine/Scene/GameObject/GameObject.Network.cs
updated: 2026-04-25
created: 2026-04-27
---

# Spawn a Prefab

Learn how to dynamically spawn prefabs into the world from your C# code, both locally and across the network.

## Quick Working Example

```csharp
public sealed class SpawnerComponent : Component
{
	[Property] public GameObject PrefabToSpawn { get; set; }

	protected override void OnUpdate()
	{
		if ( Input.Pressed( "Attack1" ) && PrefabToSpawn != null )
		{
			// Clone the prefab locally
			var spawnedObj = PrefabToSpawn.Clone();

			// Position it slightly in front of the spawner
			spawnedObj.WorldPosition = GameObject.WorldPosition + GameObject.WorldRotation.Forward * 50f;

			// If building a multiplayer game, tell the network to spawn it for everyone
			spawnedObj.NetworkSpawn();
		}
	}
}
```

### Spawning Over the Network

When you call `Clone()` on a prefab, it only exists on the machine that called it. For multiplayer games, you need to call `NetworkSpawn()` immediately after configuring the cloned object.

```csharp
public void DropItem()
{
	// Clone the item from the prefab template
	var droppedItem = ItemPrefab.Clone();

	// Set its position and rotation
	droppedItem.WorldTransform = GameObject.WorldTransform;

	// Spawn it across the network
	droppedItem.NetworkSpawn();
}
```

### Spawning with a Specific Owner

In a multiplayer context, some objects (like player characters or vehicles) need a specific client to be the "owner" so they can control them. You can pass the client's `Connection` to `NetworkSpawn()`.

```csharp
public void SpawnPlayer( Connection clientConnection )
{
	var player = PlayerPrefab.Clone();
	
	// Spawn the object and assign the specific client as the owner
	player.NetworkSpawn( clientConnection );
}
```

### Parent the Spawned Prefab

If you want the spawned prefab to follow another object (like a particle effect attached to a moving car), you can use `Clone( Transform, GameObject )` to set its initial transform and parent simultaneously.

```csharp
// Clone the prefab, placing it at the parent's position and making it a child
var attachment = EffectPrefab.Clone( parentObject.WorldTransform, parentObject );
```

## Troubleshooting

:::danger "My prefab is null when trying to spawn!"
Ensure that the `[Property] public GameObject PrefabToSpawn { get; set; }` has been assigned in the Editor. If the field is left empty in the Inspector, trying to call `.Clone()` on it will throw a `NullReferenceException`.
:::

:::warning "Other players can't see the spawned object!"
If an object spawns fine for you but not for others in a multiplayer game, you likely forgot to call `NetworkSpawn()` on the cloned `GameObject`, or the prefab isn't configured with a networked mode.
:::

## Related Pages
- [GameObject](../scene/gameobject.md)
- [Cloning](../scene/cloning.md)
- [Clone GameObjects](clone-gameobjects.md) — the lower-level recipe behind prefab spawning.
- [Spawn Particles](spawn-particles.md) — a common application of the spawn-a-prefab pattern.
- [Networked Objects](../systems/networking-multiplayer/networked-objects.md)
