---
title: "Events and Timing"
created: 2026-11-04
updated: 2026-11-11
---

# Events and Timing

Event-query nodes let the graph react to events sampled from animation. For example, a transition can wait for a planted foot or a marked interruption window instead of using an arbitrary delay.

They return Bool, Float, or ID nodes to connect inside the graph. They do not register C# callbacks.

For choosing and authoring clip events, typed C# receivers, and the difference between ID, foot, native, and graph events, see [Animation Events](/animation/animgraph/events.md).

## Gate a transition on a planted foot

Assume `machine`, `moving`, and `stopped` are already built states, and `stopRequested` is a Bool parameter:

```csharp
var leftFootDown = builder.FootstepCondition(
    GraphBuilder.FootPhase.LeftFootDown,
    new GraphBuilder.EventQuerySettings
    {
        SourceState = moving,
        IgnoreInactiveEvents = true
    }
);

machine.AddTransition(
    moving,
    stopped,
    builder.And( stopRequested, leftFootDown ),
    new GraphBuilder.TransitionSettings { Duration = 0.15f }
);
```

The moving animation needs authored footstep events. Without appropriate events, this condition cannot tell when the foot is down. Make sure every action has a reachable exit rather than relying on a marker absent from some clips.

## Query scope

`EventQuerySettings.SourceState` scopes queries to a particular state. When it is supplied, `LimitSearchToSourceState` defaults to `true`; without a source it defaults to `false`. `IgnoreInactiveEvents` excludes inactive sampled events.

During blends, several clips or states can contribute events. Choose a scope deliberately so an unrelated layer cannot satisfy the condition.

| Setting enum | Choices |
|---|---|
| `EventPriority` | `HighestWeight` (default) or `HighestPercentageThrough` |
| `EventMatch` | `Or` for any match, `And` for all listed matches in the same update |
| `EventSearch` | `SearchAll`, `OnlySearchGraphEvents`, `OnlySearchAnimEvents` |

## Footstep queries

| Method | Output |
|---|---|
| `FootstepCondition( phase, settings )` | Bool: the requested phase is sampled |
| `FootstepPercentage( phase, priority, settings )` | Float progress through that phase, or `-1` when absent |
| `FootstepEventId( priority, settings )` | ID from the selected sampled foot event |

`FootPhase` provides `LeftFootDown`, `LeftFootPassing`, `RightFootDown`, `RightFootPassing`, `LeftPhase`, `RightPhase`, and `None`. `LeftPhase` combines right-foot-down/left-foot-passing; `RightPhase` combines left-foot-down/right-foot-passing. `None` represents a marked section without a clear foot phase.

## ID and graph events

| Method | Output |
|---|---|
| `IdEventCondition( eventIds, match, search, settings )` | Bool matching sampled IDs |
| `IdEvent( defaultValue, priority, settings )` | Highest-priority sampled ID animation event, or the supplied fallback |
| `IdEventPercentage( eventId, priority, settings )` | Float progress through the matching ID animation event, or `-1` |
| `GraphEventCondition( conditions, match, settings )` | Bool matching `(EventId, Type)` graph events |

`GraphEventType` provides `Entry`, `FullyInState`, `Exit`, `Timed`, `Generic`, and `Any`.

Graph events come from state settings or a clip node's `GraphEvents`. Animation events are authored on clips. Do not confuse an event ID with an ID control parameter: one reports sampled animation data, the other is an input supplied by gameplay.

An event can remain sampled over multiple updates. Treat conditions as graph values, not automatically debounced one-shot notifications.

## Curve events

`CurveEvent` samples a float curve authored as an animation event:

```csharp
var handWeight = builder.CurveEvent( new GraphBuilder.CurveEventSettings
{
    EventId = "hand_ik",
    DefaultValue = 0,
    IgnoreInactiveEvents = true
} );
```

Feed the result into an IK or layer weight. The clip must contain the corresponding curve event; otherwise the fallback is used.

`CurveEventSettings` includes `EventId`, `DefaultValue`, optional Float `DefaultValueNode`, `Priority`, `LimitSearchToSourceState`, and `IgnoreInactiveEvents`. The node fallback takes precedence over the constant. Unlike `EventQuerySettings`, these settings do not accept an explicit `SourceState`; a source-limited query uses its transition evaluation context.

## Transition permission events

`TransitionEventCondition( rule, markerId, settings )` reads authored transition-permission events. Use it to combine a gameplay request with a marked interruption window.

`TransitionRule` provides `AnyAllowed` (default), `FullyAllowed`, `ConditionallyAllowed`, and `Blocked`. Supply `markerId` to filter to a particular marker.

This is a condition; it does not create a transition route or make all transitions obey events automatically.

## Synchronization queries

These queries require a source state and are intended for **state-machine transition conditions**:

| Method | Output |
|---|---|
| `SyncEventCondition( state, index, trigger )` | Bool comparing the current sync event index |
| `CurrentSyncEventId( state )` | Current sync event ID |
| `CurrentSyncEvent( state, info )` | Current sync position as a float |

`SyncEventTrigger` provides `ExactlyAtEventIndex` and `GreaterThanEqualToEventIndex`. The index must be nonnegative.

`SyncEventInfo` provides `IndexAndPercentage` (default), `IndexOnly`, and `PercentageOnly`. Sync-event progress is not necessarily the same as normalized progress through the whole clip.

For straightforward action completion, prefer [Completed or TimeCondition](/animation/animgraph/runtime-graph-builder/state-machines.md) rather than introducing synchronization queries unnecessarily.
