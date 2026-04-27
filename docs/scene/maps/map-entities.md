---
title: "Map Entities"
icon: "📂"
created: 2026-04-25
updated: 2026-04-27
sources:
  - engine/Sandbox.Engine/Editor/Hammer/HammerAttributes.cs
  - engine/Sandbox.Engine/Scene/Components/Map/MapObjectComponent.cs
  - engine/Sandbox.Tools/GameData/MapClass.cs
  - engine/Sandbox.Tools/GameData/MapClassType.cs
  - engine/Sandbox.Tools/GameData/EntityParser.cs
  - engine/Sandbox.Tools/GameData/GameData.Assembly.cs
  - engine/Sandbox.Tools/MapEditor/HammerEntities/CommentEntity.cs
---

# Map Entities

> **New to maps?** The Maps overview is at **[Maps](index.md)**. For authoring (the **Mapping** tool), see **[Scene Mapping](../scene-mapping/index.md)**. This page is the reference for what map entities are at runtime and how to define new ones.

A "map entity" is a placeable object inside a Hammer map — a point entity (light, prop, spawn marker), a solid entity (a brush volume tied to mesh geometry), or a path/cable entity. At runtime, every map entity becomes a child `GameObject` of the `MapInstance` that loaded the map.

This page covers two things:

1. **What runtime form a map entity takes** — every entity gets a `MapObjectComponent`, which carries the `SceneObject`s and tags from Hammer.
2. **How to define a new map entity** — by declaring a class with `[HammerEntity]` and `[Library("classname")]`, derived from `HammerEntityDefinition`.

## Built-in Entity Types

Most common Hammer entity classes are recognized by `MapInstance` directly and lowered to s&box scene components:

| Entity class | Becomes |
|---|---|
| `info_player_start` | `SpawnPoint` component |
| `prop_dynamic` / `prop_animated` | `Prop` component (static) |
| `prop_physics` | `Prop` component (networked) |
| `func_brush` | `ModelRenderer` + optional `ModelCollider` |
| `env_sky` | 2D Skybox |
| `skybox_reference` | 3D Skybox (`MapSkybox3D`) |
| `env_gradient_fog` | `GradientFog` component |
| `env_cubemap_fog` | `CubemapFog` component |
| `env_cubemap` / `env_cubemap_box` | `EnvmapProbe` component |
| `env_volumetric_fog_volume` | `VolumetricFogVolume` component |
| `snd_soundscape` | `SoundscapeTrigger` (sphere) |
| `snd_soundscape_box` | `SoundscapeTrigger` (box) |
| Lights (`light_*`) | `SceneLight` objects (directional, spot, omni, rect, capsule) |

Entity classes not in that list don't lose their data — they get a `MapObjectComponent` carrying the raw scene objects and tags. From there you can either query them yourself, or override `OnCreateObject` (next section) to give them their own component.

## Custom Entity Handling: `OnCreateObject`

To handle a custom entity class — or to override how a built-in one is created — subclass `MapInstance` and override `OnCreateObject`:

```csharp
public class MyMapInstance : MapInstance
{
    protected override void OnCreateObject( GameObject go, MapLoader.ObjectEntry kv )
    {
        if ( kv.TypeName == "ammo_pickup" )
        {
            var pickup = go.Components.Create<AmmoPickup>();
            pickup.Amount = kv.GetValue<int>( "amount", 30 );
            pickup.RespawnDelay = kv.GetValue<float>( "respawn_delay", 15f );
            pickup.Model = kv.GetResource<Model>( "model" );
            return;
        }
        // Fall through for anything else — unrecognized entities still get
        // a default MapObjectComponent unless you handle them explicitly.
    }
}
```

`OnCreateObject` is called only for entity types **not already handled internally** (the table above). So overriding it is purely additive — you don't risk breaking spawn-point or prop_physics behaviour.

### Reading keyvalues: `MapLoader.ObjectEntry`

Each entity arrives as a `MapLoader.ObjectEntry` with both raw data and typed accessors:

| Field / Method | Description |
|---|---|
| `kv.TypeName` | Entity class name from Hammer (e.g. `"prop_physics"`). |
| `kv.TargetName` | The entity's `targetname`. |
| `kv.ParentName` | The entity's `parentname`. |
| `kv.Position` / `kv.Angles` / `kv.Transform` | Spawn transform from Hammer. |
| `kv.Tags` | Tags set in Hammer. |
| `kv.GetValue<T>( key, default )` | Typed keyvalue lookup with a fallback. |
| `kv.GetString( key )` | Raw string keyvalue. |
| `kv.GetResource<T>( key )` | Resolves a `Model`, `Material`, `Soundscape`, etc. directly. |

Built-in handlers use the same API — see `MapInstance.cs:523` (`CreateStaticModel`) for a real-world example pulling `model`, `rendercolor`, etc.

## Runtime Form: `MapObjectComponent`

When `MapInstance` loads a map, every child `GameObject` it creates gets a `MapObjectComponent` (`engine/Sandbox.Engine/Scene/Components/Map/MapObjectComponent.cs`):

- The component owns a `List<SceneObject>` containing the renderable geometry for that entity (typically one for a prop, multiple for a solid brush entity).
- It copies tags from each `SceneObject` onto the parent `GameObject.Tags`, so you can tag-query map entities like any other GameObject.
- On disable, all owned `SceneObject`s are deleted; on re-enable, `RecreateMapObjects` rebuilds them.
- If a map entity fails to construct (no scene objects), the component sets `GameObjectFlags.Error` so the editor highlights it.

You generally do **not** instantiate `MapObjectComponent` yourself — it's a runtime-internal type. You interact with map entities by querying `MapInstance.GameObject.Children`, by tag, or via component lookup on a specific child:

```csharp
foreach ( var child in mapInstance.GameObject.Children )
{
    if ( child.Tags.Has( "spawnpoint" ) )
        spawnPoints.Add( child.WorldPosition );
}
```

## Defining a Hammer Entity

A Hammer entity is a C# class declared in your project's editor assembly (or in `Sandbox.Tools` for built-in entities). The minimum recipe:

```csharp
namespace MyGame.Editor;

[Library( "info_spawnpoint" )]   // The class name as it appears in Hammer's FGD
[HammerEntity]                    // Mark it as a placeable entity
[Title( "Spawn Point" )]          // Display name in Hammer's entity browser
[Icon( "place" )]                 // Material icon
[EditorSprite( "editor/spawnpoint.vmat" )] // Sprite shown in the 3D view
class SpawnpointEntity : HammerEntityDefinition
{
    [Property, FGDType( "team" )]
    public int Team { get; set; }

    [Property]
    public bool Enabled { get; set; } = true;
}
```

Once compiled, the entity:

- Appears in Hammer under the "Spawn Point" label, in whatever `[Category(...)]` you specified.
- Has its `Team` and `Enabled` keyvalues editable in Hammer's entity properties panel.
- Will be spawned at runtime as a child of the `MapInstance` with `Tags` matching whatever it specified in the `.vmap`.

### Attribute reference

| Attribute | Purpose |
|---|---|
| `[HammerEntity]` | Required. Marks the class as placeable in Hammer. Defined in `Sandbox.Engine/Editor/Hammer/HammerAttributes.cs`. |
| `[Library( "classname" )]` | Required. The Hammer/FGD class name (typically lowercase with underscores, e.g. `info_spawnpoint`). |
| `[Title( "Display Name" )]` | Display name in the Hammer entity tool. |
| `[Icon( "material_icon" )]` | Material icon for the entity browser. |
| `[Category( "Logic" )]` | Group entities under a category in the entity browser. |
| `[EditorSprite( "editor/foo.vmat" )]` | Sprite drawn in the 3D viewport at the entity's position. |
| `[Solid]` | Marks the class as a brush entity (mesh-tied) rather than a point entity. |
| `[Property]` | Standard property attribute — exposes the value to Hammer as a keyvalue. |
| `[FGDType( "team" )]` | Hints the FGD type so Hammer renders an appropriate editor (e.g. team picker, color picker, target name). |

### Class types

The kind of entity is determined by what you derive from and which attributes you apply:

| Type | Marker | Notes |
|---|---|---|
| Point | `[HammerEntity]` on a `HammerEntityDefinition` subclass | Default. Single position + rotation. |
| Solid (brush) | Add `[Solid]` | Tied to a mesh in Hammer. Geometry is in the `.vmap`, not the C# definition. |
| Path | `[PathNode]` (separate attribute) | Appears in Hammer's Path Tool. |
| Cable | `[CableClass]`-derived | Path with cable rendering. |

The full enum lives in `engine/Sandbox.Tools/GameData/MapClassType.cs` (`GameDataClassType`). Most game code only ever needs the point and solid forms.

## Editor-Side Reflection: `MapClass`

Hammer reads each `[HammerEntity]` class via `EntityParser.cs` and `GameData.Assembly.cs`, building a `MapClass` (`engine/Sandbox.Tools/GameData/MapClass.cs`) that tools query for:

- `MapClass.Name` — the class name (`info_spawnpoint`)
- `MapClass.DisplayName`, `Description`, `Icon`, `Category`
- `MapClass.Type` — the `System.Type` of the C# class
- `MapClass.IsPointClass` / `IsSolidClass` / `IsPathClass` / `IsCableClass`
- `MapClass.Variables` — `List<MapClassVariable>` exposing each `[Property]`

You generally don't touch `MapClass` directly — it's the editor's view of what Hammer should show. Game code interacts with the runtime spawn (the `GameObject` + `MapObjectComponent`) instead.

## Example: a complete custom point entity

```csharp
using Editor;          // for HammerEntityDefinition (in editor assembly)
using Sandbox;

[Library( "ammo_pickup" )]
[HammerEntity]
[Title( "Ammo Pickup" )]
[Icon( "ammo" )]
[Category( "Pickups" )]
[EditorSprite( "editor/ammo.vmat" )]
class AmmoPickupEntity : HammerEntityDefinition
{
    [Property] public int Amount { get; set; } = 30;
    [Property, FGDType( "studio" )] public string Model { get; set; } = "models/ammo_box.vmdl";
    [Property] public bool RespawnAfterPickup { get; set; } = true;
    [Property] public float RespawnDelay { get; set; } = 15f;
}
```

A runtime component on the spawned GameObject can then read these by querying the saved keyvalues — typically by adding a regular `Component` to the prefab spawned for this entity, with `[Property]` fields matching the entity definition.

## Common Patterns

1. **Mark up entities, not behavior.** A `HammerEntityDefinition` describes *what to expose to Hammer*. The actual gameplay behavior lives in regular `Component`s attached to the spawned `GameObject`. Many games pair an entity definition with a same-named component that consumes its keyvalues.
2. **Keep editor and runtime assemblies separate.** `[HammerEntity]` lives in the `Editor.MapEditor.EntityDefinitions` namespace by convention. Game runtime code lives in your normal assembly. This lets the editor strip entity definitions from the standalone build.
3. **Use existing Hammer entities first.** Built-in entities under `engine/Sandbox.Tools/MapEditor/HammerEntities/` cover lights, sound emitters, cameras, comments, environmental lighting, etc. Look there before defining a new one.

- **Entity attribute placement.** `[HammerEntity]` is internal to `Sandbox` — your class must be in an assembly that the engine compiles as part of the editor pipeline. Game-runtime-only assemblies cannot define new Hammer entities.
- **Renaming a class.** Changing `[Library("classname")]` will break existing maps that reference the old name. Treat the library name as a stable identifier.
- **Solid vs point.** A class without `[Solid]` is a point entity. Adding `[Solid]` means it can only be placed via "tie to entity" on existing brush geometry — Hammer won't let you place it as a standalone point.

## Related Pages

- [Maps overview](index.md)
- [Loading Maps](loading-maps.md)
- [Map Networking](map-networking.md)
- [Hammer (mapping)](../../editor/hammer.md)
- [Editor Tools](../../editor/editor-tools/index.md)
