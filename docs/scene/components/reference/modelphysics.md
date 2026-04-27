---
title: "Model Physics"
icon: "☐"
sources:
  - engine/Sandbox.Engine/Scene/Components/Collider/ModelPhysics.cs
  - engine/Sandbox.Engine/Scene/Components/Collider/ModelPhysics.PhysicsCreate.cs
  - engine/Sandbox.Engine/Scene/Components/Collider/ModelPhysics.Networking.cs
updated: 2026-04-25
created: 2026-04-27
---

# Model Physics

> **New to physics?** Physics overview is at **[Physics](../../../systems/physics/index.md)**. This page is the reference for the `ModelPhysics` component.

`ModelPhysics` builds a jointed physics skeleton from a model's authored physics data and drives a `SkinnedModelRenderer`'s bones from the simulated bodies. This is how ragdolls work. For a single rigid body you should use a `Rigidbody` instead — `ModelPhysics` is specifically for models with multiple parts wired together with joints (hinge, ball, fixed, slider).

## Quick Working Example

```csharp
public class RagdollOnDeath : Component
{
    [RequireComponent] SkinnedModelRenderer Renderer { get; set; }

    public void Ragdoll()
    {
        var physics = AddComponent<ModelPhysics>();
        physics.Model = Renderer.Model;
        physics.Renderer = Renderer;

        // Inherit pose & velocity from the animated renderer
        physics.CopyBonesFrom( Renderer, teleport: false );
    }
}
```

## Common Patterns

### Spawning a ragdoll from an animated character

`CopyBonesFrom( source, teleport )` reads bone transforms and velocities from another `SkinnedModelRenderer` and seeds the physics bodies with them. This is how you make a character "fall over" smoothly when killed — the ragdoll inherits the animation pose and momentum.

```csharp
var physics = go.Components.Create<ModelPhysics>();
physics.Model = renderer.Model;
physics.Renderer = renderer;
physics.CopyBonesFrom( renderer, teleport: false );
```

### Freezing a ragdoll into pose

Set `MotionEnabled = false` to switch every body into kinematic mode — they will follow animation rather than simulate.

```csharp
modelPhysics.MotionEnabled = false; // bodies follow animation
modelPhysics.MotionEnabled = true;  // bodies simulate freely
```

### Putting a ragdoll to sleep

If you spawn a pile of corpses, set `StartAsleep = true` so they don't tick the physics solver until something disturbs them. This calls `body.Sleeping = true` on every body in `OnStart`.

## Properties

| Property | Default | Description |
|---|---|---|
| `Model` | `null` | The model whose `PhysicsGroupDescription` is used to build bodies, shapes, and joints. Reassigning rebuilds physics. |
| `Renderer` | `null` | The `SkinnedModelRenderer` whose bones receive transform overrides from physics. Auto-found via `GetComponent<SkinnedModelRenderer>()` in `OnAwake` if unset. |
| `IgnoreRoot` | `false` | If true, the root body does not drive this component's `WorldTransform`. |
| `RigidbodyFlags` | `(none)` | `RigidbodyFlags` propagated to every body. |
| `Locking` | `(none)` | `PhysicsLock` (axis locks) propagated to every body. |
| `StartAsleep` | `false` | Sleep all bodies in `OnStart`. |
| `MotionEnabled` | `true` | True: physics drives the renderer (ragdoll). False: animation drives kinematic bodies (e.g., for a player who hasn't died yet). |

## Read-only state

| Property | Description |
|---|---|
| `Mass` | Sum of every body's mass. |
| `MassCenter` | Mass-weighted center of mass in world space. |
| `Bodies` | The created `Rigidbody` parts. |
| `Joints` | The created joint components. |

## Methods

- `CopyBonesFrom( SkinnedModelRenderer source, bool teleport )` — copy bone transforms and velocities from another renderer into the physics bodies. Useful for ragdoll-from-animation handoff.

:::warning "ModelPhysics vs Rigidbody"
A `ModelPhysics` component creates a `Rigidbody` per physics part on the bone's GameObject — it is not itself a single rigid body. Use a plain `Rigidbody` for things like crates and balls; reach for `ModelPhysics` only when the model has authored physics parts and joints (ragdolls, jointed props).
:::

:::danger "Model has no physics parts"
If `Model.Physics` is null or empty, `ModelPhysics` silently does nothing. The model needs to have physics shapes and joints baked in.
:::

## Related Pages

- [Skinned Model Renderer](skinnedmodelrenderer.md)
- [Spring Joint](springjoint.md)
- [Hitboxes](hitboxes.md)
