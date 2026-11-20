---
title: "Schedules & Tasks"
icon: "📋"
created: 2026-11-20
updated: 2026-11-20
---

# Schedules & Tasks

A schedule describes what an NPC is doing. A task is one step in that schedule.

`WanderSchedule`, `InvestigateSchedule`, `FollowSchedule` and the other built-in schedules cover common behaviours. Return one from `SelectSchedule()`:

```csharp
protected override BaseSchedule SelectSchedule()
{
    if ( Senses?.Disturbance is { } disturbance )
    {
        var investigate = GetSchedule<InvestigateSchedule>();
        investigate.TargetPosition = disturbance.Position;
        return investigate;
    }

    return GetSchedule<WanderSchedule>();
}
```

`GetSchedule<T>()` creates one instance per NPC and reuses it. Set its inputs each time you return it. Don't create a new schedule on every selection.

# Interruptions

When an NPC notices something new, such as an enemy or a disturbance, it can select a more important schedule. The new schedule must have a higher `Priority`, and the current one must be `Interruptible`.

Put the most important choices first in `SelectSchedule()`. The engine doesn't choose between all your schedules for you. It also selects again when a schedule finishes or fails.

# Your own schedule

Build the task list in `OnStart()`:

```csharp
using Sandbox;
using Sandbox.Npcs.Schedules;
using Sandbox.Npcs.Tasks;

public sealed class VisitSchedule : BaseSchedule
{
    public GameObject Target { get; set; }

    protected override void OnStart()
    {
        AddTask( new MoveToTask( Target, stopDistance: 80f ) );
        AddTask( new LookAtTask( Target ) );
        AddTask( new WaitTask( 2f ) );
    }

    protected override void OnEnd()
    {
        Npc.Animation.ClearLookTarget();
    }
}
```

`MoveToTask` accepts a position or a GameObject. With a GameObject, it updates the destination as the target moves.

For a custom step, derive from `BaseTask`. Its `OnUpdate()` returns `TaskResult.Running`, `Success` or `Failed`. Success advances to the next task; failure ends the schedule. Use `OnEnd()` to clean up, including when the task is interrupted.
