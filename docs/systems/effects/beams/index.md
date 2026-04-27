---
title: "Beams"
icon: "⚡"
created: 2025-06-26
updated: 2026-04-12
---

# Beams

Beams are animated line renderers ideal for creating lasers, lightning, tracers, and visual links between objects.

## Quick Working Example

```csharp
public class LaserPointer : Component
{
	[Property] public GameObject BeamPrefab { get; set; }
	BeamEffect _beamEffect;

	protected override void OnStart()
	{
		// Spawn a GameObject that has a BeamEffect component
		var go = BeamPrefab.Clone();
		_beamEffect = go.Components.Get<BeamEffect>();
	}

	protected override void OnUpdate()
	{
		if ( _beamEffect == null ) return;

		// The beam will start at this object
		_beamEffect.WorldPosition = WorldPosition;

		// And draw a line to wherever the trace hits
		var tr = Scene.Trace.Ray( WorldPosition, WorldPosition + WorldRotation.Forward * 1000f )
			.IgnoreGameObjectHierarchy( GameObject )
			.Run();

		_beamEffect.TargetPosition = tr.HitPosition;
	}
}
```

## Navigation Hub

* [**BeamEffect Component**](beameffect.md) - The core component for configuring beams, scrolling textures, and scaling.
* [**Tracers**](tracers.md) - How to use the "Travel Between Points" setting to create fast-moving projectile trails.
