---
title: "GameObject"
icon: "📦"
created: 2023-11-14
updated: 2024-10-03
sources:
  - engine/Sandbox.Engine/Scene/GameObject/GameObject.cs
  - engine/Sandbox.Engine/Scene/GameObject/GameObject.Clone.cs
  - engine/Sandbox.Engine/Scene/GameObject/GameTransform/GameTransform.cs
---

# GameObject

A `GameObject` is a thing in your scene. It has a transform (position, rotation, scale), a name, a list of tags, a list of components, and a list of children. That's basically all of it. Everything interesting comes from the components attached to it.

```csharp
var enemy = new GameObject( true, "Enemy" );
enemy.WorldPosition = new Vector3( 0, 0, 10 );
enemy.Tags.Add( "hostile" );

var renderer = enemy.Components.Create<ModelRenderer>();
renderer.Model = Model.Load( "models/citizen/citizen.vmdl" );
```

The `true` in the constructor is the initial `Enabled` flag. There are several constructor overloads — you can pass a parent, a name, or just defaults — see `GameObject.cs:101-144`.

## Position, rotation, scale

These live directly on the GameObject:

```csharp
GameObject.WorldPosition = new Vector3( 100, 0, 0 );
GameObject.WorldRotation = Rotation.FromYaw( 90 );
GameObject.WorldScale    = new Vector3( 2, 2, 2 );

// Local variants are relative to the parent
GameObject.LocalPosition = new Vector3( 0, 50, 0 );
GameObject.LocalRotation = Rotation.From( 0, 45, 0 );
```

`WorldTransform` and `LocalTransform` give you the whole thing as a `Transform` struct if you'd rather pass it around.

> The legacy `Transform.Position` / `Transform.Rotation` properties on the `GameTransform` instance are marked `[Obsolete]`. Use `WorldPosition` and friends directly on the GameObject. Code using the obsolete form still works but the compiler will warn.

## Tags

Tags are inherited by children. Common uses: filtering physics collisions, categorizing objects for queries.

```csharp
GameObject.Tags.Add( "player" );

if ( GameObject.Tags.Has( "player" ) ) { ... }

// Removal: only from this object's own tags. Inherited tags
// from parents are reported via Has() but can't be removed here —
// remove them from the parent instead.
GameObject.Tags.Remove( "player" );
```

`Tags.Has` returns true if either this object's own tag set or any ancestor's set contains the tag.

## Hierarchy

```csharp
foreach ( var child in GameObject.Children )
{
    Log.Info( child.Name );
}

// Set parent. Pass keepWorldPosition: false to keep local offsets instead.
child.SetParent( newParent );
```

A few rules worth knowing:

- `GameObject.Children` is a real `List<GameObject>` but **don't add or remove from it directly** — use `child.SetParent(parent)` or `child.Destroy()`.
- Destroying a parent destroys its children.
- A disabled parent halts everything in its subtree. Children's individual `Enabled` flags are preserved, so re-enabling the parent doesn't accidentally re-enable children that were explicitly turned off.

## Components

```csharp
var rb = GameObject.Components.Create<Rigidbody>();          // always make a new one
var rd = GameObject.Components.Get<ModelRenderer>();         // get if it exists, else null
var col = GameObject.Components.GetOrCreate<BoxCollider>();  // get or add

// Walk descendants
var allLights = GameObject.Components.GetAll<PointLight>( FindMode.EverythingInSelfAndDescendants );
```

There's also `AddComponent<T>()` as a thin wrapper over `Components.Create<T>()` if you prefer that name.

See [Components Overview](components/index.md) for the full lifecycle and patterns.

## Cloning

```csharp
// Duplicate this GameObject and its whole subtree
var copy = GameObject.Clone();

// Clone with a transform
var spawn = enemyPrefab.Clone( new Transform( spawnPosition ) );

// Clone with parent + initial transform
var attached = effectPrefab.Clone( WorldTransform, parentObject );
```

When cloning a *prefab*, you typically have a `GameObject` reference to the prefab's root and call `.Clone(...)` on it. There are also static `GameObject.Clone(string prefabPath)` overloads for loading a prefab by path.

For multiplayer, follow the clone with `NetworkSpawn()` — see [Networked Objects](../systems/networking-multiplayer/networked-objects.md).

## Validity

References to a destroyed GameObject don't auto-null. Check before use:

```csharp
if ( storedReference.IsValid() )
{
    // safe to use
}
```

This matters most in async code that yielded — the object you held a reference to may have been destroyed while you were awaiting.

## Property table

| Member | Type | Notes |
|---|---|---|
| `Name` | `string` | Display name in the scene tree |
| `Enabled` | `bool` | Master toggle for this object and its descendants' tick/render |
| `WorldPosition` / `LocalPosition` | `Vector3` | Direct on the GameObject |
| `WorldRotation` / `LocalRotation` | `Rotation` | Quaternion under the hood |
| `WorldScale` / `LocalScale` | `Vector3` | Per-axis scale |
| `WorldTransform` / `LocalTransform` | `Transform` | The whole pose at once |
| `Tags` | `GameTags` | `Add`, `Remove`, `Has`, `Contains` |
| `Children` | `List<GameObject>` | Read-only view; mutate via `SetParent` |
| `Parent` | `GameObject` | Setter calls `SetParent` |
| `Components` | `ComponentList` | The component list (Create/Get/GetAll/etc.) |
| `Scene` | `Scene` | The scene this object belongs to |
| `Network` | `NetworkObject` | Networking surface — see Networked Objects |

## Related Pages

- [Components](components/index.md)
- [Transform](transform.md) — quaternion math, world vs local, parent hierarchies
- [Cloning](cloning.md) — full details on `Clone()` overloads and clone configs
- [Tags](tags.md) — how tags compose with physics and rendering filters
