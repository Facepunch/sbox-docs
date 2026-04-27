---
title: Trace / Raycast
icon: 🍳
sources:
  - engine/Sandbox.Engine/Scene/Scene/Scene.Trace.cs
updated: 2026-04-25
created: 2026-04-27
---

# Trace / Raycast

> **New to physics?** Physics overview is at **[Physics](../systems/physics/index.md)**. This page is the recipe for one specific task.

How to fire a trace (raycast) into the scene to detect objects or world geometry.

## Quick Working Example

```csharp
using Sandbox;

public sealed class LaserSight : Component
{
    [Property] public float MaxDistance { get; set; } = 1000f;

    protected override void OnUpdate()
    {
        var startPos = GameObject.WorldPosition;
        var forward = GameObject.WorldRotation.Forward;
        var endPos = startPos + forward * MaxDistance;

        // Perform the trace
        var tr = Scene.Trace.Ray( startPos, endPos )
            .IgnoreGameObject( GameObject ) // Don't hit ourselves
            .Run();

        if ( tr.Hit )
        {
            // We hit something!
            Log.Info( $"Hit: {tr.GameObject.Name} at {tr.HitPosition}" );
            
            // Draw a debug line
            Gizmo.Draw.Line( startPos, tr.HitPosition );
        }
        else
        {
            // Didn't hit anything, draw to max distance
            Gizmo.Draw.Line( startPos, endPos );
        }
    }
}
```

### Shape Tracing

A simple ray is infinitely thin, which makes it poor for detecting players with a projectile. You can use a shape like a Sphere or Box to perform a "thick" trace.

```csharp
// Sweep a 10-unit radius sphere forward
var tr = Scene.Trace.Sphere( 10f, startPos, endPos )
    .IgnoreGameObject( GameObject )
    .Run();
```

Other available shapes include `Box`, `Capsule`, and `Cylinder`.

### Filtering by Tags

You can narrow down what your trace hits by requiring or excluding specific tags.

```csharp
// Only hit objects tagged as "enemy" or "glass"
var tr = Scene.Trace.Ray( startPos, endPos )
    .WithAnyTags( "enemy", "glass" )
    .Run();

// Hit anything EXCEPT objects tagged "water"
var tr2 = Scene.Trace.Ray( startPos, endPos )
    .WithoutTags( "water" )
    .Run();
```

### Filtering GameObjects

When firing a raycast from a weapon or player, you usually want to ensure you don't hit the player holding it.

```csharp
// Ignore the specific game object
var tr = Scene.Trace.Ray( startPos, endPos )
    .IgnoreGameObject( myPlayerObject )
    .Run();
```

## Configuration

When calling `.Run()`, you receive a `SceneTraceResult` struct containing detailed information about what was hit.

| Property | Description |
|:---|:---|
| `Hit` | `true` if the trace collided with something. |
| `HitPosition` | The exact `Vector3` point in the world where the impact happened. |
| `Normal` | A `Vector3` pointing straight away from the surface that was hit. |
| `Fraction` | A value `0.0` to `1.0` representing how far along the path the hit occurred. |
| `GameObject` | The `GameObject` that was hit. |
| `Component` | The `Component` on the PhysicsBody that was hit. |
| `Collider` | The `Collider` that was hit. |
| `Surface` | The physical `Surface` material hit, useful for spawning decals and sound effects. |
| `StartedSolid` | `true` if the trace started inside a solid object. |
| `Distance` | The distance between the trace's start and end positions. |

## Troubleshooting

:::warning Why isn't my raycast hitting anything?
A trace checks against **physics colliders**, not visual models. If you are trying to shoot an object and your raycast passes through it, ensure the target has a `Collider` component attached (e.g., `BoxCollider`, `SphereCollider`).
:::

:::danger Hitting yourself
If your trace starts at `GameObject.WorldPosition` and immediately hits the object firing it, it will return `Fraction = 0`. Always use `.IgnoreGameObject( GameObject )` to skip your own colliders.
:::

## Related Pages
- [Physics](../systems/physics/index.md)
- [Colliders](../systems/physics/colliders.md)
- [Apply Physics Forces](apply-physics-forces.md) — use the trace result to push the hit object.
