---
title: "NPCs"
icon: "🤖"
created: 2026-11-20
updated: 2026-11-20
---

# NPCs

`BaseNpc` gives you health, movement, sensing and a way to organize behaviour. Your NPC decides what to do, and the components on its GameObject do the work.

Use `BaseCombatNpc` if you want an NPC that fights with [weapons](combat.md).

# Making an NPC

Subclass `BaseNpc` and choose a schedule. This one wanders:

```csharp
using Sandbox.Npcs;
using Sandbox.Npcs.Schedules;

public sealed class Wanderer : BaseNpc
{
    protected override BaseSchedule SelectSchedule()
    {
        return GetSchedule<WanderSchedule>();
    }
}
```

Put it on a GameObject with:

* A **SkinnedModelRenderer**, assigned to the NPC's `Renderer` property.
* A **NavigationLayer** and **NavMeshAgent** for walking. Enable the scene's [NavMesh](../navigation/index.md).
* A **Collider** for hits and collisions, and a **Rigidbody** for physics interactions.
* A **CitizenNpcAnimator** if the model uses the citizen animgraph. Other models need their own animator.

The **AnimationLayer** is added automatically. Add a **SensesLayer** if the NPC needs to notice players, other NPCs or disturbances, and a **SpeechLayer** if it talks. The layers live in `Sandbox.Npcs.Layers`.

Save the GameObject as a prefab and place it in your scene. Turn on **Show Debug Overlay** to see its current schedule and layer state while testing.

# Behaviour

A [schedule](behaviour.md) is a sequence of tasks: walk somewhere, look at something, wait. The NPC selects a schedule, and the engine runs its tasks in order.

You can also override `Think()` and drive the NPC yourself. Schedules are optional. Thinking, sensing and movement run on the host; network-spawn the NPC when creating it in a multiplayer game.
