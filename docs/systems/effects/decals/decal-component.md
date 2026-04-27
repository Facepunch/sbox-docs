---
title: "Decal Component"
icon: "✴️"
created: 2025-06-26
updated: 2026-04-12
sources:
  - engine/Sandbox.Engine/Scene/Components/Render/Decal.cs
---

# Decal Component

The `Decal` component creates a projected decal relative to its GameObject, effectively painting a texture onto the surrounding models and terrain.

## Quick Working Example

```csharp
public class GraffitiPlacer : Component
{
    [Property] public DecalDefinition MyGraffiti { get; set; }

    protected override void OnUpdate()
    {
        if ( Input.Pressed( "attack1" ) )
        {
            var tr = Scene.Trace.Ray( WorldPosition, WorldPosition + WorldRotation.Forward * 500f )
                .IgnoreGameObjectHierarchy( GameObject )
                .Run();

            if ( tr.Hit )
            {
                // Create a new GameObject to hold the decal
                var decalObject = new GameObject( true, "Graffiti" );
                decalObject.WorldPosition = tr.HitPosition;
                decalObject.WorldRotation = Rotation.LookAt( tr.Normal ); // Face outward

                // Add the Decal component and assign the definition
                var decal = decalObject.Components.Create<Decal>();
                decal.Decals.Add( MyGraffiti );
                
                // Set the size to be 64x64 units wide/tall, projecting 16 units deep
                decal.Size = new Vector2( 64f, 64f );
                decal.Depth = 16f;
            }
        }
    }
}
```

### Randomizing Visuals

The component is set up so you can create variety in your prefab decals without any extra coding. Instead of a single definition, you add a list of `DecalDefinitions` to the component.

```csharp
// The engine will pick one of these at random when the object is enabled
decal.Decals.Add( BloodSplat1 );
decal.Decals.Add( BloodSplat2 );
decal.Decals.Add( BloodSplat3 );
```

Additionally, properties like `Rotation` and `Scale` use a `ParticleFloat` type. This means you can set them to be a random range. For example, setting rotation to a range from `0` to `360` ensures that every time the decal spawns, it chooses a random angle, avoiding obvious repetition in bullet impacts.

### Limiting Decals (Transient)

If you spawn a decal every time a bullet hits a wall, you will eventually crash the game due to having thousands of GameObjects. The `Transient` property solves this.

```csharp
decal.Transient = true;
```

When `Transient` is true, the decal will automatically be removed by the `DecalGameSystem` when the global max decal limit is exceeded. 

### Fading Over Time

You can define how long the decal exists before fading away by setting `LifeTime`.

```csharp
decal.LifeTime = 10f; // Lives for 10 seconds before being cleaned up
```

## Configuration

| Property | Description |
|---|---|
| `Decals` | A list of `DecalDefinition` assets. The component will choose one at random to display. |
| `Size` | A 2D vector defining the width and height of the decal in world units. |
| `Depth` | How far the decal extends into the surface it is projected onto along the local forward axis. |
| `LifeTime` | How long the decal should live for before fading out and disappearing. |
| `Looped` | If true, the decal's animation/lifetime will repeat forever. |
| `Transient` | If true, this decal will automatically get removed when global maximum decals are exceeded (good for bullet impacts). |
| `Scale` | Multiplies the width and height. Can be a curve or random range. |
| `Rotation` | Rotation angle of the decal in degrees. Can be a random range (e.g., 0 to 360). |
| `Parallax` | Parallax depth strength of the decal. |
| `ColorTint` | Tints the color of the decal's albedo. |
| `SortLayer` | Determines the order the decal gets rendered in (higher layer = draws on top). |

## Troubleshooting

:::warning Decal not showing up?
Ensure the `Decal` component is aimed correctly. It projects down the local Forward (`+X`) axis. If you just placed it facing flat against a wall, it might be projecting into the air! You usually want the rotation to use `Rotation.LookAt( hitNormal )` or equivalent so it faces back *away* from the wall.
:::

:::danger Memory Leaks with Spawning Decals
If you manually spawn `GameObject`s containing `Decal` components from code, remember that `Transient` only removes the component when the limit is reached. To ensure the entire `GameObject` is properly deleted when its `LifeTime` ends, make sure you also add a `TemporaryEffect` component to the GameObject. `Decal` natively supports the `ITemporaryEffect` interface.
:::

## Related Pages
- [DecalDefinition](decaldefinition.md)
- [Temporary Effect](../temporaryeffect.md)
