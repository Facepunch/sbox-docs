---
title: "State Machines"
created: 2026-11-04
updated: 2026-11-04
---

# State Machines

A state machine answers "what am I doing now?" Each state produces a pose; transitions decide when to change state and how to blend the change.

Use one for actions that start, finish, and return to another behavior. Do not use an entry-only selector as a continuously switching state machine.

## Idle, wave, return

This factory takes two valid clips bound to the same skeleton. It returns a graph you must retain and eventually dispose using the [ownership rules](/animation/animgraph/runtime-graph-builder/index.md).

```csharp
using Sandbox.Animation;

public static Graph CreateWaveGraph( Clip idleClip, Clip waveClip )
{
    var builder = Graph.Builder;
    builder.Skeleton = idleClip.SkeletonName;

    var request = builder.BoolParameter( "b_wave" );

    var machine = builder.StateMachine();
    var idle = machine.AddState( builder.Clip( idleClip,
        new GraphBuilder.ClipSettings { SampleRootMotion = false } ) );
    var wave = machine.AddState( builder.Clip( waveClip,
        new GraphBuilder.ClipSettings
        {
            Looping = false,
            SampleRootMotion = false
        } ) );

    var transition = new GraphBuilder.TransitionSettings
    {
        Duration = 0.2f
    };

    machine.AddTransition( idle, wave, request, transition );
    machine.AddTransition(
        wave, idle, machine.Completed( wave, 0.2f ), transition
    );

    return builder.Create( machine, "idle_and_wave" );
}
```

The first added state is the default: idle. Setting `b_wave` true starts the transition to wave. The return transition starts `0.2` seconds before completion so its blend finishes at the clip's end.

`Completed( wave )` instead uses a zero lead time: it waits until completion before starting the return blend. The builder does not automatically copy the transition duration into this condition.

## Clear the request in game code

**The graph does not reset `b_wave`.** If it stays true, the action can start again when idle is reached.

For a normally updating renderer, this small driver turns calls to `Wave()` into one-update pulses:

```csharp
using Sandbox;

public sealed class WaveRequestExample : Component
{
    [Property] public SkinnedModelRenderer Renderer { get; set; }

    private bool pending;

    public void Wave()
    {
        pending = true;
    }

    protected override void OnUpdate()
    {
        Renderer.Animation.Parameters.Set( "b_wave", pending );
        pending = false;
    }

    protected override void OnDisabled()
    {
        pending = false;

        if ( Renderer.IsValid() && Renderer.Animation.IsValid )
            Renderer.Animation.Parameters.Set( "b_wave", false );
    }
}
```

Assign a renderer running the wave graph. Call `Wave()` from gameplay; the following update clears the input unless another request arrives. Setting true and false back-to-back in the same method would not give the graph an opportunity to observe true.

This simple driver assumes the graph evaluates between component updates. If your game suspends or throttles animation evaluation, retain requests until an animation evaluation can consume them rather than discarding them on a wall-clock timer.

This graph does not queue or restart a wave while already waving. Decide those rules explicitly for your game. For runtime-selected one-shots without a bool request, use the [external clip player](/animation/animgraph/runtime-graph-builder/external-graphs.md).

## States and entry

| Method | Purpose |
|---|---|
| `StateMachine()` | Begin a machine |
| `AddState( child, ... )` | Add a state containing a pose |
| `AddOffState( ... )` | Add a no-pose state for disabling an optional layer |
| `SetDefaultState( state )` | Choose a different initial fallback state |
| `SetEntryCondition( state, condition )` | Choose the entry state when the machine initializes |
| `Build()` | Get the machine's Pose node; repeated calls return the same node |

Entry conditions select an initial state; they are not ongoing transition routes. State handles belong to their machine and cannot be used with another machine. A machine supports at most `StateMachineBuilder.MaxStates` states, currently 64.

An Off state is for [optional layers](/animation/animgraph/runtime-graph-builder/layers-and-masks.md), not a replacement for a full-body idle pose.

## Transition settings

`AddTransition( from, to, condition, settings )` creates a route taken when its Bool condition is true. Add competing routes in the order you want them considered.

All of these settings belong to `GraphBuilder.TransitionSettings`:

| Setting | Purpose |
|---|---|
| `Duration` | Blend time in seconds; default `0.2` |
| `DurationOverride` | Optional Float node replacing that duration |
| `BlendEasing` | Weight curve; default `Linear` |
| `TimeMode` | `None`, `Synchronized`, or `MatchSourceTime` |
| `TimeOffset`, `TimeOffsetOverride` | Constant or Float-node target offset, normally measured in sync events |
| `OffsetTimeInSeconds` | Use a seconds-based offset for an unsynchronized transition |
| `MatchTimeInSeconds` | Match source time in seconds when using time matching |
| `SyncMatch` | Combine `EventIndex`, `EventId`, and `EventPercentage` matching flags |
| `TargetSyncId` | Optional ID node selecting a target sync event |
| `PreferClosestSyncEventId` | Prefer the closest matching sync ID |
| `ClampDuration` | Shorten the blend if the source would finish before the transition does |
| `StartBoneMask` | Optional initial mask for a staged per-bone transition |
| `BoneMaskBlendPercentage` | Portion of transition used to blend the mask; default `0.33` |
| `RootMotionMode` | `Blend` (default), `Additive`, `IgnoreSource`, or `IgnoreTarget` |
| `CanBeForced` | Allow this route to interrupt an in-progress transition |

Leave advanced timing settings at their defaults until you need them. Synchronization aligns animation timing; it is not a generic improvement to every transition. A wave or reload usually should not inherit a walk cycle's phase.

For a synchronized transition, time offsets use whole sync-event indices. Unsynchronized sync-time offsets can also include progress through an event. These are not automatically seconds or a percentage through the whole clip.

For shared transitions, call `CreateTransition( target, settings )`, then add routes with `AddTransition( from, to, transitionNode, new TransitionRouteSettings { Condition = ..., CanBeForced = ... } )`. Use the same target as the transition node. Most graphs are clearer with the ordinary overload.

## Timing conditions

`machine.Completed( state, transitionDuration, transitionDurationOverride )` accepts an optional Float duration override. When a transition uses a dynamic duration, supply that same node to its completion condition so the lead time agrees.

`machine.TimeCondition( state, settings )` supports more specific checks:

```csharp
var lateEnough = machine.TimeCondition( wave,
    new GraphBuilder.StateMachineBuilder.TimeConditionSettings
    {
        Type = GraphBuilder.StateMachineBuilder.TimeComparisonType.PercentageThroughState,
        Comparison = GraphBuilder.StateMachineBuilder.TimeComparison.GreaterThanEqual,
        Value = 0.8f
    } );
```

`TimeComparisonType` provides `PercentageThroughState`, `PercentageThroughSyncEvent`, `ElapsedTime`, and `TimeInState`. Percentages use `0` to `1`; elapsed time and time in state use seconds. Playback time and time spent in a state can differ when playback speed or looping changes.

`TimeComparison` provides `LessThan`, `LessThanEqual`, `GreaterThan`, and `GreaterThanEqual`. `TimeConditionSettings.InputValue` optionally replaces the constant `Value`.

`builder.StateCompletedCondition` and `builder.TimeCondition` expose the same operations directly from the parent builder.

## State events and layer data

The simple `AddState` and `AddOffState` overloads accept `entryEvents`, `exitEvents`, and `fullyInStateEvents`. For more control, use `AddState( child, StateSettings )`:

| `StateSettings` | Purpose |
|---|---|
| `EntryEvents`, `FullyInStateEvents`, `ExitEvents` | Graph event IDs sampled in the corresponding state phase |
| `ElapsedEvents`, `RemainingEvents` | Timed entries as `(Id, Time, Operator)` |
| `UseActualElapsedTimeInStateForTimedEvents` | Base elapsed events on time spent in the state rather than playback timing |
| `IsOffState` | Make this an Off state |
| `LayerWeight`, `LayerRootMotionWeight`, `LayerBoneMask` | State-specific layer values blended during transitions |

`TimedEventComparison` provides `LessThanEqual` and `GreaterThanEqual`. Timed state events are conditions on elapsed/remaining time, not a promise of a one-time C# callback. See [Events and Timing](/animation/animgraph/runtime-graph-builder/events-and-timing.md).
