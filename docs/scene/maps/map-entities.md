---
title: "Map Entities"
icon: "🏗️"
created: 2026-04-11
updated: 2026-04-11
---

# Map Entities

When a map is loaded, its embedded entities are converted into GameObjects as children of the `MapInstance`'s GameObject. The following entity types are handled automatically:

| Entity Type | What Gets Created |
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
| Lights | `SceneLight` objects (directional, spot, omni, rect, capsule) |

Any entity type not listed above will get a `MapObjectComponent` that manages its scene objects directly.


# Custom Entity Handling

You can create your own `MapInstance` subclass and override `OnCreateObject` to handle custom entity types or modify how built-in types are created. This method is only called for entity types that are not already handled internally.

```csharp
public class MyMapInstance : MapInstance
{
    protected override void OnCreateObject( GameObject go, MapLoader.ObjectEntry kv )
    {
        if ( kv.TypeName == "my_custom_entity" )
        {
            var myComponent = go.Components.Create<MyComponent>();
            myComponent.Health = kv.GetValue<float>( "health", 100 );
            myComponent.SpawnDelay = kv.GetValue<float>( "spawn_delay", 0 );
        }
    }
}
```


# ObjectEntry

The `ObjectEntry` struct gives you access to all the entity's key-value data from Hammer.

```csharp
kv.TypeName      // Entity class name, e.g. "prop_physics"
kv.TargetName    // The entity's target name
kv.ParentName    // The entity's parent name
kv.Position      // World position
kv.Angles        // Rotation angles
kv.Transform     // Full transform (position, rotation, scale)
kv.Tags          // Tags set in Hammer
```

You can read typed values, strings, and resource references from the entity's key-value pairs.

```csharp
// Get a typed value with a default fallback
float health = kv.GetValue<float>( "health", 100 );
Color color = kv.GetValue<Color>( "rendercolor", Color.White );

// Get a string value
string message = kv.GetString( "message" );

// Get a resource reference
Model model = kv.GetResource<Model>( "model" );
Material mat = kv.GetResource<Material>( "material" );
```

# Entity Parenting

Map entities that have a `parentname` set in Hammer will be parented to the corresponding entity's GameObject. Entities are sorted during load so that parents are created before their children.
