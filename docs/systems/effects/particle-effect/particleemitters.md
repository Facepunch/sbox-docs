---
title: "Particle Emitters"
icon: "⭕"
sources:
  - engine/Sandbox.Engine/Scene/Components/Particles/Emitter/ParticleEmitter.cs
  - engine/Sandbox.Engine/Scene/Components/Particles/Emitter/ParticleBoxEmitter.cs
  - engine/Sandbox.Engine/Scene/Components/Particles/Emitter/ParticleConeEmitter.cs
  - engine/Sandbox.Engine/Scene/Components/Particles/Emitter/ParticleModelEmitter.cs
  - engine/Sandbox.Engine/Scene/Components/Particles/Emitter/ParticleRingEmitter.cs
  - engine/Sandbox.Engine/Scene/Components/Particles/Emitter/ParticleSphereEmitter.cs
created: 2026-04-27
updated: 2026-04-27
---

# Particle Emitters

Particle Emitters dictate where, when, and how fast particles are created within a `ParticleEffect`.

## Quick Working Example

Usually you configure emitters in the Scene Editor. If you want to modify an emitter from code:

```csharp
using Sandbox;

public sealed class EmitterController : Component
{
    [Property] public ParticleBoxEmitter BoxEmitter { get; set; }

    protected override void OnUpdate()
    {
        if ( BoxEmitter == null ) return;

        if ( Input.Pressed( "attack1" ) )
        {
            // Reset the emitter to trigger its Initial Burst again
            BoxEmitter.ResetEmitter();
        }
    }
}
```

### Bursts vs Rates

Emitters have two main ways to spawn particles:
- **Burst:** Spawns a specific number of particles instantly when the emitter starts. Useful for explosions, sparks, or single-shot effects.
- **Rate:** Spawns a continuous stream of particles over time. Useful for fire, smoke, rain, or continuous magic beams.

You can mix these! An explosion effect might have a burst of sparks, followed by a rate of smoke that lasts a few seconds.

### Looping and Destruction

Emitters can be set to **Loop** (continuously restart their duration and bursts) or run exactly once. If they do not loop, you can enable **DestroyOnEnd** to automatically delete the GameObject when emission finishes and all particles have died out.

## Types of Emitters

There are several built-in emitter shapes:

*   **Box Emitter:** Emits particles randomly within a configurable 3D box. Useful for rain or snow volumes.
*   **Sphere Emitter:** Emits particles within a sphere, optionally only on the outer edge, and can apply outward velocity. Useful for explosions.
*   **Cone Emitter:** Emits particles in a cone shape with a set angle and radius. Ideal for flashlights, thrusters, or directional spells.
*   **Ring Emitter:** Emits particles along a 2D ring, with options to offset them. Useful for shockwaves.
*   **Model Emitter:** Emits particles across the surface of a `ModelRenderer` or `SkinnedModelRenderer`. Great for character auras or burning objects.

## Configuration

These base properties apply to all emitters:

| Property | Description |
|:---|:---|
| `Loop` | Whether the emitter should restart after finishing its `Duration`. |
| `DestroyOnEnd` | Whether to destroy the GameObject when the emitter finishes (only applies when `Loop` is false). |
| `Duration` | How long the emitter should run for, after the `Delay`. |
| `Delay` | How many seconds to wait before the emitter starts. |
| `Burst` | How many particles to emit in a sudden burst when starting. |
| `Rate` | How many particles to emit per second over time. |
| `RateOverDistance` | How many particles to emit per 100 units moved. Useful for trails. |

## Troubleshooting

:::warning Emitter not spawning anything?
Ensure the GameObject has a `ParticleEffect` component. Emitters cannot function without a parent effect to hold the particles. Also, check that your `Rate` or `Burst` values are above 0.
:::

## Related Pages
- [Particle Effect](particleeffect.md)
- [Spawn Particles](../../../how-to/spawn-particles.md)
