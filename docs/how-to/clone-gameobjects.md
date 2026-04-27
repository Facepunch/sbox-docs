---
title: "Clone GameObjects"
icon: "📋"
sources:
  - engine/Sandbox.Engine/Scene/GameObject/GameObject.Clone.cs
updated: 2026-04-25
created: 2026-04-27
---

# Clone GameObjects

Quickly spawn exact copies of GameObjects or Prefabs at runtime.

## Quick Working Example

```csharp
using Sandbox;

public sealed class ProjectileSpawner : Component
{
    [Property] public GameObject ProjectilePrefab { get; set; }
    [Property] public Transform SpawnPoint { get; set; }

    protected override void OnUpdate()
    {
        if ( Input.Pressed( "Attack1" ) && ProjectilePrefab != null )
        {
            // Clone the prefab and set its position and rotation in one go
            GameObject clone = ProjectilePrefab.Clone( SpawnPoint );
            
            // Or if it's a networked multiplayer game, make sure everyone sees it
            // clone.NetworkSpawn();
        }
    }
}
```

### Spawning at a Specific Position

You can pass a `Transform` (which includes Position, Rotation, and Scale) or just a `Vector3` position directly into the `Clone()` method.

```csharp
// Clone at a specific 3D coordinate
Vector3 spawnPosition = new Vector3( 0, 0, 100 );
GameObject bullet = ProjectilePrefab.Clone( spawnPosition );

// Clone with full transform matching a specific marker
GameObject enemy = EnemyPrefab.Clone( SpawnMarker.WorldTransform );
```

### Spawning as a Child

If you want the cloned object to immediately become a child of another GameObject (for example, spawning a weapon attachment that follows the player's hand), pass the parent object as the second argument.

```csharp
// The clone will be parented to 'PlayerHand' and inherit its world transform
GameObject weapon = WeaponPrefab.Clone( PlayerHand.WorldTransform, PlayerHand );
```

### Advanced Cloning Configuration

For more complex spawning rules, you can configure a `CloneConfig` struct and pass it to `Clone()`.

```csharp
var config = new CloneConfig
{
    Transform = new Transform( new Vector3( 0, 50, 0 ) ),
    StartEnabled = false, // Spawn it disabled so we can set it up before it runs
    Name = "Stealth Enemy",
    Parent = EnemyContainer
};

GameObject customClone = EnemyPrefab.Clone( config );

// Do some extra setup here...

// Now enable it
customClone.Enabled = true;
```

## Troubleshooting

:::danger "Object spawns at the origin (0,0,0)!"
If you call `GameObject.Clone()` without providing a `Transform` or `Vector3`, the new object will spawn at `Vector3.Zero`. Always provide a transform if you want it to appear at a specific location.
:::

:::warning "Clone is instantly destroyed!"
If you clone an object whose original is currently in the process of being destroyed (e.g., inside an `OnDestroy` callback), the cloning may fail or the clone may also be destroyed. Only clone from valid, active GameObjects or stable Prefab files.
:::

## Related Pages
- [Cloning Concept](../scene/cloning.md)
- [Spawn a Prefab](spawn-prefab.md)
- [Spawn Particles](spawn-particles.md) — a common case of cloning a prefab at runtime.
