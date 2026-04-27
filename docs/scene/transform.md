---
title: "Transform"
icon: "🎮"
created: 2024-05-15
updated: 2024-05-15
sources:
  - engine/Sandbox.Engine/Scene/GameObject/GameTransform/GameTransform.cs
  - engine/Sandbox.Engine/Scene/GameObject/GameObject.WorldTransform.cs
  - engine/Sandbox.Engine/Scene/GameObject/GameObject.LocalTransform.cs
---

# Transform

Where a GameObject sits in the world: position, rotation, scale. You access these directly on the GameObject — there is no separate `Transform` component to attach.

```csharp
GameObject.WorldPosition += GameObject.WorldRotation.Forward * 10f * Time.Delta;
GameObject.WorldRotation *= Rotation.FromYaw( 90f * Time.Delta );
```

## World vs local

Every transform property has two flavours:

- **World** — relative to the scene's absolute origin. `WorldPosition`, `WorldRotation`, `WorldScale`.
- **Local** — relative to the parent GameObject. `LocalPosition`, `LocalRotation`, `LocalScale`.

If a GameObject has no parent, the two are identical. With a parent, modifying local moves the child *with* the parent's frame; modifying world positions it absolutely.

> A person standing inside a train: their `LocalPosition` is where they're standing inside the carriage; their `WorldPosition` is where the carriage is on the map. Move the train and `WorldPosition` changes, `LocalPosition` doesn't.

## Setting position, rotation, scale together

If you're updating multiple parts at once, build a `Transform` struct and assign it once instead of writing to each property in turn:

```csharp
GameObject.LocalTransform = new Transform(
    position: new Vector3( 0, 0, 50 ),
    rotation: Rotation.FromYaw( 90f ),
    scale:    2.0f
);
```

Same for `WorldTransform`. Slightly faster than three separate writes because the engine recomputes derived state once instead of three times.

## Visual interpolation

`OnFixedUpdate` runs at fixed rate (~50 Hz). `OnUpdate` and rendering run at framerate. Without interpolation, an object moved in `OnFixedUpdate` would appear to teleport between physics ticks at high refresh rates.

The engine handles this automatically: rendering uses an interpolated transform between the previous and current physics-tick positions. You don't need to manage it. If you genuinely need the interpolated value (custom render pass, debug overlay), it's at:

```csharp
var visualPos = GameObject.Transform.InterpolatedLocal.Position;
```

But this is rare. Stick to `WorldPosition` etc. for almost everything.

## Reference

| Member | Type | Notes |
|---|---|---|
| `WorldPosition` | `Vector3` | Absolute world position |
| `WorldRotation` | `Rotation` | Absolute world rotation (quaternion) |
| `WorldScale` | `Vector3` | Absolute world scale |
| `LocalPosition` | `Vector3` | Relative to parent |
| `LocalRotation` | `Rotation` | Relative to parent |
| `LocalScale` | `Vector3` | Relative to parent |
| `WorldTransform` | `Transform` | The full world pose as one struct |
| `LocalTransform` | `Transform` | The full local pose as one struct |

## Things that bite

**`Transform.Position` is obsolete.** Older tutorials show `GameObject.Transform.Position` (capital T, the GameTransform instance). It still compiles but the engine marks it `[Obsolete]`. Use `GameObject.WorldPosition` directly.

**Setting `WorldPosition` on a dynamic Rigidbody teleports it.** The physics integrator owns the transform of dynamic bodies; manual writes don't fight you, they just teleport the body. If you want smooth physics motion, set `Rigidbody.Velocity` instead, or use `Rigidbody.SmoothMove()` if you're moving toward a target. To make a body Kinematic (you drive it, physics doesn't), set `Rigidbody.MotionEnabled = false`.

**Non-uniform scale breaks physics colliders.** Scaling a `BoxCollider`-bearing GameObject to `(1, 2, 1)` gives you a tall visual box but the collider treats it as if it were the authored size. Resize the `Collider` itself instead of scaling the GameObject.

**Quaternions don't have `.x += 1`.** `Rotation` is a quaternion under the hood. Don't mutate components directly. Use `Rotation.FromPitch`, `FromYaw`, `FromAxis`, `LookAt`, or compose with multiplication: `myRotation *= Rotation.FromYaw( 90 )`.

## Related Pages

- [GameObject](gameobject.md) — the container these properties live on
- [Math Types](../code/code-basics/math-types.md) — `Rotation` quaternion gotchas, `Vector3` operations
- [Rotate an Object (recipe)](../how-to/rotate-object.md)
