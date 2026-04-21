---
title: "Custom Particle Controller"
icon: "🎮"
created: 2023-11-27
updated: 2023-11-27
---

# Custom Particle Controller

The particle system is designed to let you control the particles via your own code based components, if you choose to.

To do this, just inherit a component from `ParticleController`.

```csharp
public class MyParticleController : ParticleController
{
	protected override void OnParticleStep( Particle particle, float delta )
	{
        // apply gravity
		particle.Velocity += Vector3.Down * 900 * delta;
	}
}
```

Then add this to your GameObject with a Particle Effect on it. You will see that the particles go up, slowly.

# OnParticleStep

This method is called for each particle in the particle system. The second argument is the time delta (which is the game time delta multiplied by the particle system's time delta).

This method is run in a thread, so you should be careful about what you're doing here. For example, don't create callbacks here, don't delete GameObjects or components etc.

# Before & After

For convenience, there are functions that are called before and after the particle step. These are always called on the main thread, so you can execute any callbacks you need to here. Here's an example.

```csharp
public class MyParticleController : ParticleController
{
	Action callbacks;

	protected override void OnBeforeStep( float delta )
	{
		callbacks = null;
	}

	protected override void OnParticleStep( Particle particle, float delta )
	{
		if ( particle.Age > 10.0f )
		{
			lock ( this )
			{
				callbacks += () => SceneUtility.Instantiate( myPrefab, particle.Position );
			}
		}
	}

	protected override void OnAfterStep( float delta )
	{
		callbacks?.Invoke();
		callbacks = null;
	}
}
```
