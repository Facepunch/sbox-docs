---
title: "Physics"
icon: "📂"
sources:
  - engine/Sandbox.Engine/Systems/Physics/
updated: 2026-04-25
created: 2026-04-27
---

# Physics

s&box uses Source 2's Rubikon physics, exposed to C# through two component types: `Rigidbody` (mass and forces) and `Collider` (shape). Every Scene has its own physics world; you don't manage it directly.

```csharp
using Sandbox;

public sealed class PhysicsPusher : Component
{
    [Property] public float PushForce { get; set; } = 1000f;

    [RequireComponent] public Rigidbody Body { get; set; }

    protected override void OnUpdate()
    {
        if ( Input.Pressed( "use" ) )
        {
            Body.ApplyImpulse( Vector3.Up * PushForce );
        }
    }
}
```

`ApplyImpulse` is for instant pushes. `ApplyForce` is for continuous force (gravity, wind). The difference matters: an impulse is mass-aware and immediate; a force needs to be applied over time.

## The two components

A `Rigidbody` without a `Collider` doesn't collide with anything — it'll fall through the world. A `Collider` without a `Rigidbody` is static collision (the floor, walls). The pairing decides what an object is:

| Has Rigidbody | Has Collider | Behaviour |
|---|---|---|
| ❌ | ✓ | Static obstacle. Other things bump into it. Doesn't move from physics. |
| ✓ | ✓ Dynamic | Falls, can be pushed, simulates fully |
| ✓ (Kinematic) | ✓ | Moves only when you set its position/velocity directly. Other dynamics react to it but it doesn't react to them. |
| ✓ | ❌ | Useless — Rigidbody with no shape, won't collide |

The single `Rigidbody.MotionEnabled` boolean decides Dynamic vs Kinematic. `true` (default) = the integrator simulates this body fully. `false` = the body holds whatever pose you write to it, but other dynamics still react to it. "Static" isn't a Rigidbody mode — it just means *no* `Rigidbody`, only a `Collider`.

Collider shapes: `BoxCollider`, `SphereCollider`, `CapsuleCollider`, `HullCollider` (a custom convex hull), `MeshCollider` (a triangle mesh — slow for moving objects). Always pick the simplest shape that fits.

## Tracing (raycasting)

When you want to *query* the physics world without affecting it (line-of-sight, hitscan weapons, ground checks), use `Scene.Trace`:

```csharp
var tr = Scene.Trace
    .Ray( WorldPosition, WorldPosition + Vector3.Down * 100f )
    .IgnoreGameObject( GameObject )    // don't hit yourself
    .Run();

if ( tr.Hit )
{
    Log.Info( $"Hit {tr.GameObject.Name} at {tr.HitPosition}" );
}
```

There are also `Sphere`, `Box`, `Capsule`, and `Cylinder` overloads that sweep a shape rather than a ray. See [Trace / Raycast](physics-events.md) for the full builder API.

## Triggers

Set `Collider.IsTrigger = true` to make a shape pass-through but still detect entry/exit:

```csharp
public class HurtZone : Component, Component.ITriggerListener
{
    public void OnTriggerEnter( Collider other )
    {
        if ( other.GameObject.Components.TryGet<PlayerHealth>( out var hp ) )
            hp.TakeDamage( 10 );
    }

    public void OnTriggerExit( Collider other ) { }
}
```

Both objects need a Collider; at least one needs a Rigidbody for events to fire. See [Collisions and Triggers](collisions-and-triggers.md).

## Where to do physics work in a frame

Use `OnFixedUpdate` for anything force-driven. It runs at a fixed rate (typically 50 Hz) regardless of framerate, so applying a constant force per fixed update produces consistent behaviour. `OnUpdate` runs at framerate and is fine for *reading* physics state (e.g. checking if the player is grounded for an animation) but not for *driving* it.

For reading the state at exactly the right moment around a physics step, implement `IScenePhysicsEvents` — see [Physics Events](physics-events.md).

## Things that catch people out

**Object falls through the floor.** Either the floor has no `Collider` (very common), the falling object is moving so fast it tunnels (enable `Rigidbody.EnhancedCcd` for continuous collision detection against dynamic bodies), or the Collider's tags exclude collision with the floor.

**Setting `WorldPosition` on a dynamic Rigidbody jitters.** The physics integrator owns the transform of dynamic bodies; manual writes fight it. Set `Body.Velocity` instead, or switch the body to Kinematic if you really want to drive it from code.

**Mesh colliders on moving objects are slow and fragile.** They work, but a moving `MeshCollider` is the most expensive collision shape there is and the engine warns about it. Substitute primitives or a `HullCollider` whenever possible.

**Non-uniform scale (e.g. squishing a sphere into an oval) doesn't work properly.** The physics engine treats colliders as their authored shape. Resize the collider's `Radius`/`HalfExtents` instead of scaling the GameObject.

**Triggers don't fire if neither object has a Rigidbody.** At least one party in a trigger interaction needs a Rigidbody. The most common pattern: trigger volume is static (Collider only); the player has both Rigidbody and Collider.

## Read next

- [Rigidbody](rigidbody.md) — every property, plus the velocity / impulse / force methods
- [Colliders](colliders.md) — shapes, materials, friction, restitution
- [Physics World](physics-world.md) — `Scene.PhysicsWorld`, gravity, fixed timestep
- [Collisions and Triggers](collisions-and-triggers.md) — `OnCollisionStart`, `OnTriggerEnter`, tag filtering
- [Joints](joints.md) — connecting bodies (hinge, slider, ball, fixed, spring)
- [Physics Events](physics-events.md) — `PrePhysicsStep` / `PostPhysicsStep`

How-to recipes: [Apply Physics Forces](../../how-to/apply-physics-forces.md), [Trace / Raycast](physics-events.md), [Collide GameObjects](../../how-to/apply-physics-forces.md).
