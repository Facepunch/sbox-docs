---
title: "Custom Particle Controller"
icon: "🎮"
created: 2023-11-27
updated: 2026-04-12
sources:
  - engine/Sandbox.Engine/Scene/Components/Particles/Controllers/ParticleController.cs
---

# Custom Particle Controller

The particle system exposes a `ParticleController` component that lets you completely control and modify the behavior of individual particles using C# logic.

## Quick Working Example

```csharp
public class MyGravityController : ParticleController
{
	protected override void OnParticleStep( Particle particle, float delta )
	{
		// Apply custom gravity to this specific particle
		particle.Velocity += Vector3.Down * 900f * delta;
	}
}
```

To use this, attach your `MyGravityController` component to the same GameObject that contains your `ParticleEffect` component. 

## Usage Patterns

### Safely Interacting with the World

`OnParticleStep` is called once for every particle, every frame. **This method is heavily multithreaded.** 

Because it runs across multiple threads, you must never interact with the main scene graph from inside `OnParticleStep`. Do not create GameObjects, delete components, or log messages directly inside it.

If you need a particle to spawn a GameObject (like an explosion prefab) when it reaches a certain age, you must capture that intent and execute it on the main thread using `OnAfterStep` (or `OnPostStep`).

```csharp
public class FireworkController : ParticleController
{
	Action callbacks;

	protected override void OnParticleStep( Particle particle, float delta )
	{
		// Check if the particle is older than 10 seconds
		if ( particle.Age > 10.0f )
		{
			// Lock the controller to safely add to our delegate from multiple threads
			lock ( this )
			{
				callbacks += () => SceneUtility.Instantiate( MyExplosionPrefab, particle.Position );
			}
		}
	}

	// This runs on the main thread after all particles have finished simulating
	protected override void OnAfterStep( float delta )
	{
		// Invoke any callbacks we queued up
		callbacks?.Invoke();
		callbacks = null;
	}
}
```

## Troubleshooting

:::danger "My game crashes when trying to spawn objects from a particle!"
If you call `GameObject.Clone()` or `SceneUtility.Instantiate()` directly inside `OnParticleStep`, you will crash the engine. `OnParticleStep` runs on a background worker thread. You must queue your actions and execute them in `OnAfterStep` which safely runs on the main thread.
:::

## Related Pages
- [Particle Effect](index.md)
