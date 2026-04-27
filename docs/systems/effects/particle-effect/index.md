---
title: "Particle Effect"
icon: "🎆"
created: 2023-11-27
updated: 2026-04-27
sources:
  - engine/Sandbox.Engine/Scene/Components/Particles/ParticleEffect.cs
---

# Particle Effect

The `ParticleEffect` component allows you to create dynamic, programmable particle systems simulated entirely on the CPU.

## Quick Working Example

You can spawn an existing particle effect from a prefab, or create and configure one entirely from C# code.

```csharp
public class MagicSpell : Component
{
	protected override void OnStart()
	{
		// 1. Add the core particle effect component
		var effect = Components.GetOrCreate<ParticleEffect>();
		effect.MaxParticles = 500;
		effect.Lifetime = 2.0f;
		effect.TimeScale = 1.0f;

		// 2. Add an emitter to spawn particles
		var emitter = Components.GetOrCreate<ParticleBoxEmitter>();
		
		// 3. Add a renderer to draw the particles
		var renderer = Components.GetOrCreate<ParticleSpriteRenderer>();
	}
}
```

## Configuration

The `ParticleEffect` component exposes numerous settings to control the simulation.

| Property | Description |
|:---|:---|
| `MaxParticles` | The maximum number of particles that can exist in this effect at once. |
| `Lifetime` | The lifetime of each particle, in seconds. |
| `TimeScale` | Multiplies the simulation speed for this entire effect. |
| `LocalSpace` | If set to 1.0, particles move relative to the emitter's transform rather than leaving a trail in world space. |
| `ApplyColor` | Enables the Color/Tint/Gradient module for particles. |
| `ApplyShape` | Enables the Scale/Stretch module for particles. |
| `ApplyRotation`| Enables the Pitch/Yaw/Roll rotation module for particles. |

## Troubleshooting

:::danger "My particles aren't showing up!"
Ensure your GameObject has **all three** necessary components: a `ParticleEffect`, an Emitter (like `ParticleBoxEmitter`), and a Renderer (like `ParticleSpriteRenderer`). Without a renderer, particles are invisible; without an emitter, no particles are spawned.
:::

## Related Pages
- [Custom Particle Controller](custom-particle-controller.md)
