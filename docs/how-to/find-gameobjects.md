---
title: "Find GameObjects"
icon: "🔍"
sources:
  - engine/Sandbox.Engine/Scene/Scene/Scene.cs
  - engine/Sandbox.Engine/Scene/Scene/GameObjectDirectory.cs
created: 2026-04-27
updated: 2026-04-27
---

# Find GameObjects

Learn how to search the scene hierarchy to find GameObjects by name, ID, or tags.

## Quick Working Example

```csharp
using Sandbox;
using System.Linq;

public sealed class EnemyManager : Component
{
	protected override void OnStart()
	{
		// Find a specific GameObject by name
		var boss = Scene.Directory.FindByName( "Boss_Dragon" ).FirstOrDefault();

		if ( boss != null )
		{
			Log.Info( $"Found the boss at {boss.WorldPosition}" );
		}
	}
}
```

### Finding by Name
`FindByName` returns an enumerable of all GameObjects with a matching name. It is case-insensitive by default.

```csharp
// Get the first object named "SpawnPoint"
var spawnPoint = Scene.Directory.FindByName( "SpawnPoint" ).FirstOrDefault();

// Get ALL objects named "Coin"
var allCoins = Scene.Directory.FindByName( "Coin" );
```

### Finding by Tags
You can search the scene for objects that have specific tags. This is often more robust than searching by name.

```csharp
// Find all objects tagged "enemy"
var enemies = Scene.FindAllWithTag( "enemy" );

// Find all objects that have BOTH "enemy" and "boss" tags
var bossEnemies = Scene.FindAllWithTags( new[] { "enemy", "boss" } );
```

### Iterating All Objects
If you need to process every single object in the scene, you can use `Scene.GetAllObjects()`. 

```csharp
// Iterate over all enabled objects in the scene
foreach ( var go in Scene.GetAllObjects( true ) )
{
	// Do something with each object
}
```

### Finding Components Directly
If you are looking for a GameObject because you want to interact with a specific Component on it, it is usually better to find the Component directly instead.

```csharp
// Find the first PlayerController component in the entire scene
var player = Scene.GetAllComponents<PlayerController>().FirstOrDefault();

if ( player != null )
{
	// Now you have the component AND its GameObject
	var playerGo = player.GameObject;
}
```

## Troubleshooting

:::warning FindByName returns an IEnumerable
`Scene.Directory.FindByName` does not return a single `GameObject`. It returns an `IEnumerable<GameObject>` because multiple objects can share the same name. You must use `.FirstOrDefault()` (requiring `using System.Linq;`) or loop through the results.
:::

:::danger Performance Cost
Searching for objects (especially `FindAllWithTag` or iterating `GetAllObjects`) can be slow. Do not do this inside `OnUpdate` every frame. Instead, find the object once in `OnStart` and save it to a variable, or use `[Property]` references.
:::

## Related Pages
- [GameObject](../scene/gameobject.md)
- [Component Queries](component-queries.md)
