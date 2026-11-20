---
title: "Movement & Animation"
icon: "🚶"
created: 2026-11-20
updated: 2026-11-20
---

# Movement & Animation

Movement layers decide how an NPC gets somewhere. Schedules use `Npc.Movement`, so the same `MoveToTask` can work with a walking or flying NPC.

# Walking

Add a `NavigationLayer` and `NavMeshAgent`, then enable the scene's [NavMesh](../navigation/index.md). Configure walking and running speeds on the layer. It uses the agent for paths and avoidance, and handles [navigation links](../navigation/navmesh-links.md).

![An NPC traverses ladder, jump, drop and climb links. 1920x1080](images/navigation-links.mp4)

When driving an NPC yourself, you can request movement directly:

```csharp
Movement.MoveTo( destination, stopDistance: 32f );
```

Call `Movement.Stop()` to cancel the move.

# Flying

`FlyingLayer` moves directly toward its destination without a NavMesh. It accelerates, hovers and slides along obstacles, but it doesn't find a route around them. A blocked move can fail.

**Hover Height** sets its height above the destination. **Bounds** can keep it in air, in water, or allow both.

![Flying NPCs move without a NavMesh. 1920x1080](images/flying.mp4)

An NPC can have more than one movement layer. Enable the one it starts with, then switch on the host with `SetMovement()`:

```csharp
var flying = GetComponent<FlyingLayer>( true );
SetMovement( flying );
```

Switching stops the old move and disables the other movement layers. A running `MoveToTask` fails, allowing the NPC to select its next schedule. Derive from `MovementLayer` for another way of moving.

# Animation

`AnimationLayer` holds the NPC's movement, look target, grounded state and weapon stance. It doesn't choose animations for every model.

Use `CitizenNpcAnimator` for the citizen animgraph. For another rig, write an animator that reads the layer's state and drives the model's graph or sequences. `Animation.Triggered` carries named actions, and `BaseCombatWeapon.IEvents` carries weapon actions such as firing and reloading.

```csharp
Animation.SetLookTarget( target );
```

Set a look target when the NPC should watch something, then call `Animation.ClearLookTarget()` when that behaviour ends. A custom mover should publish its velocity through `Animation.SetMove( velocity )`. **Enable Footstep Sounds** controls the layer's footstep audio.
