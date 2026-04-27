---
title: "Particle Effect"
icon: "🚿"
sources:
  - engine/Sandbox.Engine/Scene/Components/Particles/ParticleEffect.cs
created: 2026-04-27
updated: 2026-04-27
---

# Particle Effect

The `ParticleEffect` component is the core of the particle system, responsible for defining the behavior, limits, physics, and simulation space of particles.

## Quick Working Example

While you typically configure particle effects in the Editor and spawn them from prefabs, you can fully control and create them from C# code.

```csharp
using Sandbox;

public sealed class ParticleManager : Component
{
    protected override void OnStart()
    {
        // Add the core particle effect component
        var effect = Components.GetOrCreate<ParticleEffect>();
        
        // Configure limits and simulation speed
        effect.MaxParticles = 2000;
        effect.Lifetime = 3.0f;
        effect.TimeScale = 1.0f;
        
        // Enable collision and apply a bounce factor
        effect.Collision = true;
        effect.Bumpiness = 0.5f;
        
        // Enable force to pull particles down (gravity)
        effect.Force = true;
        effect.ForceDirection = Vector3.Down;
        effect.ForceScale = 800f;

        // Remember to also add an Emitter and a Renderer!
        Components.GetOrCreate<ParticleBoxEmitter>();
        Components.GetOrCreate<ParticleSpriteRenderer>();
    }
}
```

### Spawning Prefabs on Particles
You can use a particle system to spawn actual GameObjects by enabling the Prefab feature. This is useful for leaving physical debris or dynamic lights behind.

```csharp
var effect = Components.GetOrCreate<ParticleEffect>();

// Enable the prefab feature
effect.UsePrefabFeature = true;

// Assign the prefab you want to spawn
effect.FollowerPrefab = new List<GameObject> { MyDebrisPrefab };

// Control whether the spawned prefab is destroyed when the particle dies
effect.FollowerPrefabKill = true; 
```

### Reacting to Particle Deaths
Since the simulation runs on the CPU, you can easily hook into particle events. For example, you can trigger sounds or spawn secondary effects when a particle reaches the end of its life.

```csharp
protected override void OnStart()
{
    var effect = Components.GetOrCreate<ParticleEffect>();
    
    // Subscribe to the particle destruction event
    effect.OnParticleDestroyed += OnParticleDied;
}

void OnParticleDied( Particle p )
{
    // Do something at the particle's final position
    Log.Info( $"A particle died at {p.Position} after {p.Age} seconds!" );
}
```

## Configuration

The `ParticleEffect` component offers modular feature groups. You can enable or disable entire blocks of functionality (like Color, Shape, or Collision) to optimize performance and keep the inspector clean.

### Core Properties

| Property | Description |
|:---|:---|
| `MaxParticles` | The maximum number of particles that can exist in this effect at once. Limits memory and performance overhead. |
| `Lifetime` | The lifetime of each particle, in seconds, before it is destroyed. |
| `TimeScale` | Scales the simulation speed for the entire effect (e.g., 0.5 for slow motion). |
| `Timing` | Determines if the effect runs on `GameTime` (affected by global game time scale) or `RealTime`. |
| `LocalSpace` | If set to 1.0, particles move relative to the emitter's transform rather than leaving a trail in world space. Useful for effects attached to moving objects. |

### Feature: Collision
When enabled, particles will collide with the physics world.

| Property | Description |
|:---|:---|
| `CollisionRadius` | The size of the particle used for collision detection. |
| `DieOnCollisionChance` | The probability (0 to 1) that a particle will be destroyed upon hitting a surface. |
| `Bumpiness` | How much energy the particle retains when bouncing off a surface. |
| `Friction` | How quickly the particle slows down when sliding along a surface. |
| `CollisionIgnore` | A `TagSet` specifying which physics layers the particles should ignore. |

### Feature: Force
When enabled, applies constant or orbital forces to the particles.

| Property | Description |
|:---|:---|
| `ForceDirection` | The 3D direction vector of the applied force. |
| `ForceScale` | The intensity multiplier of the force. |
| `OrbitalForce` | Applies a rotational force around the effect's center. |
| `OrbitalPull` | Draws particles closer to the center as they orbit. |
| `ForceSpace` | Whether the force direction is calculated in `Local` or `World` space. |

### Other Key Features
*   **Color:** Controls `Tint`, `Gradient`, `Brightness`, and `Alpha` over the particle's lifetime.
*   **Shape:** Modifies the physical dimensions with `Scale` and `Stretch`.
*   **Rotation:** Applies `Roll`, `Pitch`, and `Yaw` to the particle's orientation.
*   **Prefab:** Attaches a `GameObject` prefab to each particle.

## Troubleshooting

:::danger "My particles aren't spawning!"
If your `ParticleEffect` component is set up correctly but you don't see anything, check two things:
1. Did you attach an **Emitter** component (like `ParticleBoxEmitter`) to tell the effect to spawn particles?
2. Did you attach a **Renderer** component (like `ParticleSpriteRenderer`) so the particles are visible?
:::

:::warning "Particles are lagging the game!"
If you have a high `MaxParticles` count and have enabled features like **Collision**, the CPU cost can become significant. Reduce `MaxParticles`, simplify collision shapes, or consider if the effect can be achieved without physical collisions.
:::

## Related Pages
- [Particle Emitters](particleemitters.md)
- [Custom Particle Controller](custom-particle-controller.md)
- [Spawn Particles](../../../how-to/spawn-particles.md)
