---
title: "NPCs & Weapons"
icon: "🔫"
created: 2026-11-20
updated: 2026-11-20
---

# NPCs & Weapons

Mounts can provide NPC and weapon prefabs alongside models, materials and sounds. Like any prefab, they can use the engine's components or their own.

## Loading prefabs

With the mount enabled, load a prefab by its `mount://` path and clone it into your scene. In a networked game, do this on the host:

```csharp
if ( !Networking.IsHost ) return;

var prefab = PrefabFile.Load( npcPrefabPath );
var npc = GameObject.Clone( prefab, new Transform( spawnPosition ) );

npc?.NetworkSpawn();
```

## Using them in your game

How you interact with a prefab depends on its components. Those built on `BaseNpc` and `BaseCombatWeapon` use the engine's [NPC](/gameplay/npcs/index.md) and [weapon](/gameplay/inventory-weapons/weapons.md) APIs. NPCs using `NavigationLayer` need a NavMesh in your scene.

## Creating your own

See [Creating Mounts](/game-mounts/creating-mounts.md#npc-and-weapon-prefabs) for registering prefabs. The [Source mounts](https://github.com/Facepunch/sbox-mounts/tree/main/SourceEngine) are an example implementation.
