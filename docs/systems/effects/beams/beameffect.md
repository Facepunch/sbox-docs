---
title: "BeamEffect"
icon: "〰️"
created: 2025-06-26
updated: 2026-04-12
sources:
  - engine/Sandbox.Engine/Scene/Components/Effects/BeamEffect.cs
---

# BeamEffect

The `BeamEffect` component is an animated line renderer used for creating lasers, tracers, and visual links between objects.

## Quick Working Example

```csharp
public class LightningStrike : Component
{
	[Property] public Material LightningMaterial { get; set; }

	protected override void OnStart()
	{
		var beam = Components.GetOrCreate<BeamEffect>();
		beam.Material = LightningMaterial;
		
		// Configure it to spawn one thick beam
		beam.MaxBeams = 1;
		beam.Scale = 15f;
		
		// Animate the texture scrolling rapidly
		beam.TextureScrollSpeed = 10f;
		
		// Additive rendering for a glow effect
		beam.Additive = true;
	}

	protected override void OnUpdate()
	{
		var beam = Components.Get<BeamEffect>();
		if ( beam == null ) return;

		// Start the beam high in the sky
		beam.WorldPosition = new Vector3( 0, 0, 2000f );
		
		// Target the ground
		beam.TargetPosition = new Vector3( 0, 0, 0 );
		
		// Add some random noise to the endpoint so it strikes a small area
		beam.TargetRandom = 50f;
	}
}
```

### Following Points

By default, the start position of a beam is the `GameObject`'s `WorldPosition`, and the end position is the `TargetPosition`. If you enable `FollowPoints`, the beam will continuously update its positions every frame as the object moves.

If `FollowPoints` is disabled, the beam will permanently stay at the exact world coordinates where it was originally spawned, even if the source object moves away.

### Travel / Tracers

You can make the beam visually "travel" from the start point to the end point over its lifetime, which is perfectly suited for high-speed bullet tracers.

```csharp
var beam = Components.Get<BeamEffect>();
beam.TravelBetweenPoints = true;

// The beam will interpolate from start to end over its lifespan
beam.TravelLerp = new ParticleFloat { 
    Evaluation = ParticleFloat.EvaluationType.Life, 
    Type = ParticleFloat.ValueType.Range, 
    ConstantA = 0, 
    ConstantB = 1 
};
```

### Spawning Multiple Beams

You can configure the component to continuously emit new beams over time, or burst them all at once.

```csharp
var beam = Components.Get<BeamEffect>();

// Spawn 10 beams immediately
beam.InitialBurst = 10;
beam.MaxBeams = 10;
beam.BeamsPerSecond = 0; // Don't spawn any more after the burst

// Each beam lives for exactly 0.5 seconds
beam.BeamLifetime = 0.5f;
```

## Configuration

| Property | Description |
|---|---|
| `TargetGameObject` | If set, the beam will end at this GameObject's position. Overrides `TargetPosition`. |
| `TargetPosition` | The world-space end position of the beam. |
| `TargetRandom` | Adds a random offset vector to the target position, up to this magnitude. |
| `FollowPoints` | If true, active beams update their start and end points as the source/target move. |
| `MaxBeams` | The maximum number of beam instances allowed alive at once. |
| `InitialBurst` | Number of beams to instantly spawn when the component is enabled. |
| `BeamsPerSecond` | Rate of continuous beam spawning. Set to 0 to disable continuous spawning. |
| `BeamLifetime` | How long each individual beam instance lives. |
| `Scale` | The thickness/width of the beam. Can be animated over the beam's life. |
| `Material` | The material used to render the beam. |
| `TextureScale` | How many world units each texture tile covers. |
| `TextureScrollSpeed` | Speed at which the texture continuously scrolls along the beam. |
| `Additive` | If true, renders the beam additively (glowing). |

## Troubleshooting

:::danger Beams Not Disappearing
If you spawn a `BeamEffect` dynamically from code but the GameObject never gets destroyed after the beams finish, ensure you have also attached a `TemporaryEffect` component to the GameObject.
:::

:::warning Invisible Beams
If your beams are not visible, double-check that `Alpha` and `Brightness` are not set to `0`. Also, ensure that the `Material` is assigned and uses a shader compatible with line rendering (like `materials/default/default_line.vmat`).
:::

## Related Pages
- [Tracers](tracers.md)
- [Temporary Effect](../temporaryeffect.md)
