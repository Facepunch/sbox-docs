---
title: "Physics Filter"
icon: "⛔"
sources:
  - engine/Sandbox.Engine/Scene/Components/Joint/FilterJoint.cs
updated: 2026-04-25
created: 2026-04-27
---

# Physics Filter

> **New to physics?** Physics overview is at **[Physics](../../../systems/physics/index.md)**. This page is the reference for the `PhysicsFilter` component.

`PhysicsFilter` makes two physics bodies ignore collisions with each other. It's implemented as a degenerate physics joint (a "filter joint") that exists purely to suppress contact between body A (this `GameObject`) and body B (the `Body` reference) — there's no constraint, no spring, just a no-collide pair.

## Quick Working Example

Stop a thrown grenade from colliding with the player who threw it:

```csharp
var grenade = grenadePrefab.Clone();
grenade.WorldPosition = thrower.AimRay.Position;

var filter = grenade.AddComponent<PhysicsFilter>();
filter.Body = thrower.GameObject;
```

The grenade's rigidbody and the thrower's body now ignore each other for collision purposes; everything else still collides normally.

## Properties

| Property | Description |
|:---|:---|
| `Body` | The other `GameObject` to ignore collisions with. If unset, the filter is created against the scene's static world body. Reassignment rebuilds the joint. |

## Common Patterns

### Temporary no-collide

`PhysicsFilter` is just a component — add it for the duration you need the ignore, then destroy it:

```csharp
var filter = grenade.AddComponent<PhysicsFilter>();
filter.Body = thrower.GameObject;

await Task.DelaySeconds( 0.5f ); // grenade clears the thrower
filter?.Destroy();
```

### Pass-through-world objects

Leave `Body` unset to make the attached body ignore the entire static world. The fallback to `Scene.PhysicsWorld.Body` is automatic.

:::warning "One filter, one pair"
The component creates a single filter joint between exactly two bodies. To no-collide with multiple targets, add multiple `PhysicsFilter` components (each with its own `Body`).
:::

:::note "Bodies are resolved via Joint.FindPhysicsBody"
This means the filter follows the same body-resolution rules as other joint components — the body comes from the nearest `Rigidbody`/`Collider`/`MapCollider` on the `GameObject` chain.
:::

## Related Pages

- [Joints](../../../systems/physics/joints.md)
- [Spring Joint](springjoint.md)
- [Rigidbody](../../../systems/physics/rigidbody.md)
