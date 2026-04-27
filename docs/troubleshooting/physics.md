---
title: Physics & Collision
icon: "🏐"
created: 2026-04-25
updated: 2026-04-25
---

# Physics & Collision

Things falling through floors, triggers behaving like walls, characters sinking, traces missing.

## Falling through geometry

### "My Rigidbody / character / prop falls through the floor"

Almost always: one of the two objects is missing a Collider.

- **Check:** the floor has a `Collider` (BoxCollider, ModelCollider, etc.).
- **Check:** the falling object has both a Rigidbody **and** a Collider. A Rigidbody alone has no shape.
- **Fast-moving object:** enable Continuous Collision Detection on the Rigidbody.

**Deep dive:** [Rigidbody](../systems/physics/rigidbody.md), [Colliders](../systems/physics/colliders.md).

### "Players fall through my Hammer Mesh / map geometry"

The Hammer Mesh GameObject doesn't have a `ModelCollider` (or other Collider) on it. The mesh renders without colliding.

- **Fix:** add a `ModelCollider` to the GameObject containing the Hammer Mesh.

**Deep dive:** [Mapping](./mapping.md), [Hammer Mesh](../scene/components/reference/hammer-mesh.md).

### "Objects fall through my Terrain"

Terrain collision is opt-in.

- **Fix:** tick `EnableCollision` on the `Terrain` component (it's true by default — verify it wasn't unticked).

**Deep dive:** [Creating Terrain](../systems/terrain/creating-terrain.md).

### "Objects fall through MapInstance physics"

`MapInstance.EnableCollision` is off, or the source map wasn't compiled with physics.

- **Check:** `EnableCollision = true` on the MapInstance.
- **Check:** the map was compiled with physics enabled in Hammer.

**Deep dive:** [Map Instance](../scene/components/reference/mapinstance.md).

## Triggers and collisions

### "My trigger is pushing the player away — it acts like a solid wall"

The Collider's **Is Trigger** checkbox is unchecked.

- **Fix:** tick **Is Trigger** in the Inspector. Triggers are non-solid; they fire `OnTriggerEnter` instead of physically blocking.

**Deep dive:** [Collisions and Triggers](../systems/physics/collisions-and-triggers.md).

### "My OnTriggerEnter / collision events aren't firing"

Both objects need Colliders. Triggers also need an entering object capable of triggering them (it has a Collider or Rigidbody that interacts with triggers).

- **Check:** both GameObjects have Colliders.
- **Check:** the trigger has Is Trigger ticked.
- **Check:** collision tags aren't filtering each other out.

**Deep dive:** [Physics Events](../systems/physics/physics-events.md).

### "TriggerHurt isn't dealing damage"

`TriggerHurt` requires a Collider on the **same GameObject** to define its volume.

- **Fix:** add a `BoxCollider` (or similar) on the same GameObject as `TriggerHurt`.

**Deep dive:** [Trigger Hurt](../scene/components/reference/triggerhurt.md).

### "RadiusDamage hits objects through walls"

`Occlusion` is unchecked, or the wall doesn't have proper physics collision.

- **Fix:** tick `Occlusion` on `RadiusDamage`.
- **Check:** wall has a Collider that isn't tagged `trigger` or otherwise filtered out.

**Deep dive:** [Radius Damage](../scene/components/reference/radiusdamage.md).

### "RadiusDamage doesn't apply force to objects"

Force only applies to objects with a Rigidbody whose `MotionEnabled` is true (i.e., not kinematic/static).

- **Check:** target has Rigidbody with `MotionEnabled = true`.

## Character controller

### "My character is floating or sinking"

`Height` and `Radius` on the `CharacterController` don't match the visual model. The collision capsule is bigger or smaller than the model, so the character appears to float or sink.

- **Fix:** measure the model and set Height/Radius to match.

**Deep dive:** [Character Controller](../scene/components/reference/charactercontroller.md), [Player Movement](../how-to/player-movement.md).

### "My character isn't moving"

`CharacterController` is **not** a Rigidbody. `ApplyForce` does nothing.

- **Fix:** call `Controller.Move( velocity * Time.Delta )` or `Controller.MoveTo( target, ... )` from your `OnUpdate`.

### "Character stops on small bumps / can't climb stairs"

`StepHeight` is too low.

- **Fix:** raise `StepHeight` above the height of typical obstacles (a few units for normal stairs).

### "Player falls through floor with a PlayerController"

Same root cause as Rigidbody falling through: missing Colliders, or `BodyCollisionTags` excluding the floor's tag.

- **Check:** ground has a Collider.
- **Check:** `PlayerController.BodyCollisionTags` matches the tags of the ground (default is `solid`).

**Deep dive:** [Player Controller](../scene/components/reference/player-controller.md).

## Rigidbody behavior

### "I set WorldPosition every frame on a Rigidbody and it acts weird"

Setting position on a Rigidbody is a *teleport*. Physics state is invalidated each frame.

- **Fix:** apply forces (`ApplyForce`, `ApplyImpulse`) for natural movement.
- **Fix:** for kinematic-style motion, use `Rigidbody.SmoothMove`.

**Deep dive:** [Rigidbody](../systems/physics/rigidbody.md), [Apply Physics Forces](../how-to/apply-physics-forces.md).

### "ApplyForce doesn't move my object"

The Rigidbody has no mass — usually because there's no Collider.

- **Check:** Rigidbody has a Collider (mass is computed from collider volume + density).
- **Check:** `MotionEnabled = true` and not locked on the axis you're pushing.

### "My Rigidbody acts weird when I rotate it"

Setting `WorldRotation` directly teleports the Rigidbody, which the physics solver doesn't love.

- **Fix:** use `ApplyTorque`, or `SmoothMove` with a rotation.

## Joints

### "My joint doesn't do anything"

Both objects must have valid `Rigidbody` components. A joint constrains *physical* bodies.

- **Fix:** Rigidbody on both ends.

**Deep dive:** [Joints](../systems/physics/joints.md).

### "Connected bodies are spinning out of control"

`EnableCollision` on the joint is on, and the bodies are intersecting. The physics engine fights itself trying to push them apart while the joint pulls them together.

- **Fix:** disable collision between the two joint bodies, or space them apart so they don't intersect.

### "SpringJoint is vibrating or flying away"

Frequency too high relative to body mass.

- **Fix:** lower `Frequency`, or raise `Damping`.

## Tracing / Raycast

### "My raycast doesn't hit anything"

Traces hit **physics colliders**, not visual models.

- **Fix:** the target must have a `Collider` component. ModelRenderer alone is not enough.
- **Check:** the trace's `WithTag` / `WithoutTag` filters aren't excluding the target.

**Deep dive:** [Trace / Raycast](../systems/physics/physics-events.md), [Tracing](../scene/scenes/tracing.md).

### "My trace hits the object firing it"

The trace starts inside the player/weapon's own collider.

- **Fix:** `.IgnoreGameObjectHierarchy( GameObject )` to skip yourself.
- **Fix:** `.WithoutTags( "player" )` if your own colliders are tagged.

### "Scene.PhysicsWorld.Trace returns physics objects, I want GameObjects"

`PhysicsWorld.Trace` is the low-level API. It returns `PhysicsBody` / `PhysicsShape`.

- **Fix:** use `Scene.Trace.Ray(...)` instead — it gives you a result with `GameObject`.

**Deep dive:** [Physics World](../systems/physics/physics-world.md).

## Performance

### "Physics is tanking my framerate"

`SubSteps` too high, or too many active Rigidbodies.

- **Fix:** lower `SubSteps` on the Physics World (default is fine for most games).
- **Fix:** put far-away objects on a sleep/disable schedule.

## Related Pages

- [Scene & Components](./scene-and-components.md)
- [Mapping](./mapping.md)
- [Networking](./networking.md)
