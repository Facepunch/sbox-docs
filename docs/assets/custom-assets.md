---
title: "Custom Assets"
icon: "📄"
created: 2026-04-11
updated: 2026-04-11
sources:
  - engine/Sandbox.Engine/Resources/GameResource.cs
  - engine/Sandbox.Engine/Resources/GameResourceAttribute.cs
  - engine/Sandbox.Engine/Resources/ResourceLibrary.cs
---

# Custom Assets

Custom Assets let you define your own data structures that can be created in the Editor, saved to disk as `.yourext` files, and loaded at runtime.

## Quick Start

```csharp
using Sandbox;

// 1. Define your custom asset class. AssetTypeAttribute uses property syntax —
//    set Name and Extension via the attribute's named arguments.
[AssetType( Name = "Enemy Data", Extension = "enemy" )]
public class EnemyData : GameResource
{
    public string EnemyName { get; set; } = "Grunt";
    public float Health { get; set; } = 100f;
}

// 2. Load it anywhere in your game.
var grunt = ResourceLibrary.Get<EnemyData>( "data/enemies/grunt.enemy" );
Log.Info( grunt.EnemyName );
```

### Referencing assets from components

The safest way to use a `GameResource` is to expose it as a `[Property]` on a `Component`. The Editor shows an asset picker filtered to files with the matching extension.

```csharp
public class EnemySpawner : Component
{
    // Asset picker for files ending in .enemy
    [Property] public EnemyData EnemyType { get; set; }

    protected override void OnStart()
    {
        if ( EnemyType is null ) return;
        Log.Info( $"Spawning {EnemyType.EnemyName} with {EnemyType.Health} HP!" );
    }
}
```

### Loading by path

When you must load by string path (e.g. picking from a config or generating paths at runtime), use `ResourceLibrary.TryGet<T>` so a missing or wrongly-typed asset doesn't throw:

```csharp
if ( ResourceLibrary.TryGet<EnemyData>( "data/enemies/boss.enemy", out var bossData ) )
{
    SpawnBoss( bossData );
}
else
{
    Log.Warning( "Boss data not found!" );
}
```

`ResourceLibrary.Get<T>(string)` and `Get<T>(int identifier)` are the throwing variants if you'd rather fail loud.

### Enumerating every instance

To find every `GameResource` of a given type currently in the project (for spawn tables, registries, dropdowns, etc.):

```csharp
foreach ( var enemy in ResourceLibrary.GetAll<EnemyData>() )
{
    Log.Info( enemy.EnemyName );
}

// Restrict to a folder; recursive by default.
foreach ( var enemy in ResourceLibrary.GetAll<EnemyData>( "data/enemies/", recursive: true ) )
{
    ...
}
```

This walks the mounted asset registry — fast, no I/O on call.

## `[AssetType]` configuration

`AssetTypeAttribute` exposes **named properties**, not a positional constructor. Set whichever you need:

| Property | Type | Default | Description |
|---|---|---|---|
| `Name` | `string` | `null` (defaults to the class name on registration) | Display name in editor menus and asset browser. |
| `Extension` | `string` | — | File extension when saved. Must be ≤ 8 characters and letters only. The engine logs an error and falls back to `"errored"` if you violate this. |
| `Category` | `string` | `"Other"` | Group name in the asset-creation menus. |
| `Flags` | `AssetTypeFlags` | `None` | See below. |

`AssetTypeFlags`:

| Flag | Effect |
|---|---|
| `NoEmbedding` | The resource cannot be embedded inside another asset — it must exist as its own file on disk. |
| `IncludeThumbnails` | Bundles the thumbnail when this asset is published as part of a package. |

Example with everything set:

```csharp
[AssetType(
    Name      = "Enemy Data",
    Extension = "enemy",
    Category  = "Gameplay",
    Flags     = AssetTypeFlags.IncludeThumbnails )]
public class EnemyData : GameResource { ... }
```

You can also use standard property attributes (`[Property]`, `[Description]`, `[Range]`, `[Group]`, etc.) on the fields inside your `GameResource` to customise the generated inspector. See [Property Attributes](../editor/property-attributes.md).

## Migrating from `[GameResource]`

The older `[GameResource("Title", "ext", "Description")]` three-argument attribute still compiles but is `[Obsolete]`. Its `Description` field is also marked obsolete — the engine now reads the class's XML `/// <summary>` comment instead. Migrating:

```csharp
// Old (still compiles, deprecation warning)
[GameResource( "Enemy Data", "enemy", "Data for an enemy" )]
public class EnemyData : GameResource { ... }

// New
/// <summary>Data for an enemy.</summary>
[AssetType( Name = "Enemy Data", Extension = "enemy" )]
public class EnemyData : GameResource { ... }
```

## Troubleshooting

:::warning ResourceLibrary.Get returns null
If `Get` or `TryGet` fails to find your asset, check that the file exists on disk, the path is correct (relative to the mounted asset roots — usually `data/...` not the OS path), the asset has compiled successfully in the editor, and the type parameter `<T>` matches the declared `[AssetType]` class.
:::

:::danger Asset doesn't appear in the Create menu
Most common cause: the `Extension` exceeds 8 characters or contains non-letter characters. The engine logs an error to the console and substitutes `"errored"`. Check the editor console after compiling.
:::

## Related Pages

* [Assets Overview](index.md)
* [Components](../scene/components/index.md)
* [Property Attributes](../editor/property-attributes.md) — controls inspector layout for your asset's fields.
