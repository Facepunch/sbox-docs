---
title: "Tracers"
icon: "🔫"
created: 2025-06-26
updated: 2026-04-12
sources:
  - engine/Sandbox.Engine/Scene/Components/Effects/BeamEffect.cs
---

# Tracers

Learn how to use the `BeamEffect` component to create high-speed weapon tracers that fly through the air.

## Quick Working Example

You can configure a `BeamEffect` component on a GameObject to act as a tracer by enabling the `Travel` feature. This script demonstrates how to spawn a tracer prefab that automatically travels from the gun's barrel to the hit location.

```csharp
using Sandbox;

public sealed class WeaponTracer : Component
{
    [Property] public GameObject TracerPrefab { get; set; }

    public void FireTracer( Vector3 startPos, Vector3 endPos )
    {
        if ( TracerPrefab == null ) return;

        // Clone the tracer prefab at the start position
        var tracer = TracerPrefab.Clone( startPos );

        // Get the BeamEffect component from the cloned object
        var beam = tracer.Components.Get<BeamEffect>();

        if ( beam != null )
        {
            // Set the target position so the beam knows where to travel to
            beam.TargetPosition = endPos;

            // Make sure the effect will actually travel
            beam.TravelBetweenPoints = true;

            // Set a short lifetime so the tracer moves quickly
            beam.BeamLifetime = 0.1f;
        }
    }
}
```

### Setting up the Tracer Prefab

You don't typically configure tracers entirely in C#. You should create a Prefab in the Editor with a `BeamEffect` component and configure it visually:

1. Create a new GameObject and add a `BeamEffect` component.
2. In the `BeamEffect` properties, set `BeamLifetime` to a short value like `0.1`.
3. Check the `TravelBetweenPoints` box (this might be under a "Travel" feature toggle depending on your editor view).
4. Set `TravelLerp` to interpolate from `0` to `1` over the beam's life.
5. Save this GameObject as a Prefab.

Now you can spawn this prefab from your weapon script as shown in the quick example.

### Using TemporaryEffect for Cleanup

Because tracers are spawned frequently (every time a bullet is fired), you must ensure they are destroyed when they finish travelling to avoid memory leaks. Add a `TemporaryEffect` component to your tracer prefab. It will automatically detect when the `BeamEffect` has finished and destroy the GameObject.

These are the key properties on the `BeamEffect` component needed to create a tracer:

| Property | Description |
|---|---|
| `TravelBetweenPoints` | Enables the travel behavior. Must be `true` for tracers. |
| `TravelLerp` | Controls how the beam moves. For a standard tracer, this should be a `ParticleFloat` curve that goes from `0` to `1` over the particle's life. |
| `BeamLifetime` | How long (in seconds) the tracer takes to reach its destination. Lower values mean faster tracers. `0.1` is a good starting point for bullets. |
| `TargetPosition` | The world coordinate where the tracer ends up. Set this via code when you spawn the tracer. |

## Troubleshooting

:::danger "My tracer just stretches instantly from the gun to the wall!"
You forgot to enable `TravelBetweenPoints` on the `BeamEffect`. Without this, it behaves like a laser beam instead of a traveling projectile.
:::

:::warning "The tracer stays in the air and doesn't disappear!"
Make sure you have added a `TemporaryEffect` component to your tracer prefab. The `BeamEffect` itself does not destroy the GameObject when its lifetime ends.
:::

## Related Pages
- [BeamEffect](beameffect.md)
- [Particle Effect](../particle-effect/particleeffect.md)
