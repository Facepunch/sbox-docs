---
title: "Spawn Particles"
icon: "✨"
sources:
  - engine/Sandbox.Engine/Scene/Components/Particles/ParticleEffect.cs
updated: 2026-04-25
created: 2026-04-27
---

# Spawn Particles

How to spawn particle effects (like explosions, sparks, or magic spells) in your game from C#.

## Quick Working Example

You can spawn a visual particle effect from code by instantiating a new `GameObject`, adding a `ParticleEffect` component to it, and loading the visual effect resource. 

However, since visual effects are typically short-lived, it's easier to create a reusable Prefab that contains the `ParticleEffect` and the `DestroyOnStop` setting, and then spawn that Prefab.

```csharp
using Sandbox;

public sealed class MagicWand : Component
{
    // Assign a prefab in the Editor that contains your ParticleEffect
    [Property] public GameObject SparkPrefab { get; set; }

    public void CastSpell()
    {
        if ( SparkPrefab == null ) return;

        // Clone the prefab into the world at our wand's position
        var spark = SparkPrefab.Clone( GameObject.WorldPosition, GameObject.WorldRotation );
        
        // No need to clean it up manually if the Prefab's ParticleEffect has 
        // its 'Destroy Game Object' behavior set to 'On Stop'.
    }
}
```

### Changing Particle Color at Runtime

If your `ParticleEffect` uses `ParticleGradient` or references an attribute, you can adjust the look of the particle before it plays. Another common approach is adjusting the GameObject's `Tint`.

```csharp
public void CastSpell( Color spellColor )
{
    if ( SparkPrefab == null ) return;

    var spark = SparkPrefab.Clone( GameObject.WorldPosition );
    
    // If you've set up your ParticleEffect to use Tint, this will color the particles
    var effect = spark.Components.Get<ParticleEffect>();
    if ( effect != null )
    {
        effect.Tint = spellColor;
    }
}
```

## Troubleshooting

:::warning My particles won't go away!
If you clone a `ParticleEffect` prefab and it stays in your scene forever (even after the particles stop emitting), ensure you have set the `OnStop` behavior on the `ParticleEffect` component to **Destroy Game Object** in the Editor.
:::

:::danger The particles don't move with the object!
If you want a particle effect (like a fire trail) to follow a moving character, make sure you parent the cloned GameObject to the character, rather than just spawning it at the world position once.

`spark.SetParent( characterGameObject );`
:::

## Related Pages
- [Spawn a Prefab](spawn-prefab.md)
- [Clone GameObjects](clone-gameobjects.md) — the underlying clone-with-transform mechanism.
