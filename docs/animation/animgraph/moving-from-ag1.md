---
title: "Moving from AG1"
created: 2026-11-04
updated: 2026-11-04
---

# Moving from AG1

AG2 is a different graph system, not a new skin for the old editor. Reuse your animation sources and the intent of your old graph, but rebuild its behavior using the new resources and nodes.

Start with one character and one working behavior. Keep the old setup available for comparison while you migrate.

## What changes?

| In AG1 | In AG2 |
|---|---|
| `.vanmgrph` graph | `.ag` graph |
| Model sequence name used by an animation node | Separate `.anim` clip bound to a `.skel` |
| Model-authored weightlists | Bone masks authored on the `.skel` |
| `Sandbox.AnimationGraph` / renderer `AnimationGraph` | `Sandbox.Animation.Graph` / renderer `Animation.Graph` |
| Renderer `Set(...)` or `Parameters.Set(...)` | `Animation.Parameters.Set(...)` |
| `SceneModel.SetAnimParameter(...)` | `SceneModel.Animation.Parameters.Set(...)` |
| Auto-reset action/gesture parameters | Explicitly clear action bools after an animation update |
| Direct Playback node and API | An authored action state, or an External Graph slot for runtime-selected behavior |

**Assigning an `.ag` does not redirect old API calls to it.** Legacy setters still target AG1.

## Migrate the assets first

Create or reuse one skeleton, then import a small set of clips against it. Start with idle and movement before adding actions, aiming, or IK.

Preserve the old source settings that affect the result: scale, take, trim range, duration, and additive subtraction source/frame. Use [skeletons and clips](/animation/animgraph/skeletons-and-clips.md) to prepare the assets.

Build [a one-clip graph](/animation/animgraph/getting-started.md) and check it on the actual renderer before recreating the rest of the character.

## Update the parameter calls

For a graph declaring a Float named `speed`, change:

```csharp
Renderer.Set( "speed", speed );
```

to:

```csharp
Renderer.Animation.Parameters.Set( "speed", speed );
```

Update reads as well as writes. The old renderer getters and `SceneModel.GetFloat`/`GetBool` do not read AG2 parameters.

Do not assume every legacy integer parameter should remain an integer. AG2 uses **ID** parameters for named choices. For example, a graph with an ID named `holdtype` can receive:

```csharp
Renderer.Animation.Parameters.Set( "holdtype", "rifle" );
```

Audit the components writing parameters too. **CitizenAnimationHelper still uses the legacy API.** The standard PlayerController has an AG2 path, but a custom driver needs updating explicitly.

## Rebuild behavior, not the old wiring

| Behavior | A simple AG2 approach |
|---|---|
| Idle/walk/run from speed | [Blend 1D](/animation/animgraph/blending-movement.md) |
| Jump, land, or play an action | [States and transitions](/animation/animgraph/states-and-transitions.md) |
| Reload or wave while moving | [State Machine Layer with a bone mask](/animation/animgraph/layers-and-masks.md) |
| Smooth a value used by several branches | A Spring inside a virtual parameter |
| Choose one idle variation on entry | Selector |
| Change the current animation when a parameter changes | State Machine, not an entry-only Selector |

Leave independent actions unsynchronized with locomotion. Synchronizing a reload layer to the walk cycle changes its timing; it is not a general "make blending smoother" switch.

Use Overlay for ordinary poses and Additive for correctly authored additive clips. An old additive setup may depend on a particular subtraction frame, not simply on a layer checkbox.

## Replace Direct Playback deliberately

For a known action such as a wave, use a state and a request parameter. This keeps its blending and interruption rules visible in the graph.

For behavior chosen at runtime, add **External Graph** with a unique slot name such as `action`. Connect an idle or movement pose to **Fallback**, and put its output where the selected animation should appear.

```text
Locomotion ---> External Graph (action).Fallback
               External Graph.Result ---> Graph Output
```

Create a separate `.ag` containing your action clip. Both graphs must use the **same `.skel` resource**. For an available, empty slot:

```csharp
Renderer.Animation.ExternalGraphs.Connect(
    "action",
    Sandbox.Animation.Graph.Load( "graphs/wave.ag" )
);
```

When the action is finished, return to the fallback:

```csharp
Renderer.Animation.ExternalGraphs.Disconnect( "action" );
```

These are start/stop operations, not per-frame calls. Connecting to an occupied slot throws; disconnect it before replacing or restarting the action.

The switch is immediate, not an automatic crossfade. Author blending with states or layers. A non-looping clip holds its last pose until you disconnect it; completion does not empty the slot for you.

External graphs do not inherit matching parent control-parameter values. Keep this simple child graph self-contained. Connections are runtime-only and can be lost when a model or animation resource reload recreates the controller.

**External Pose is not Direct Playback.** It is for externally computed poses, not selecting an animation clip.

You do not need to author a separate `.ag` for every runtime-selected clip. The [external-graph guide](/animation/animgraph/external-graphs.md) includes a complete clip player that builds a child graph in C#, checks evaluated playback completion, and disconnects it. The [runtime state-machine guide](/animation/animgraph/runtime-graph-builder/state-machines.md) also shows an explicit one-shot bool driver.

## Check the differences that are easy to miss

Check movement while stationary, accelerating, stopping, and changing direction. Check action entry, completion, repeated requests, and interruptions. Check both the graph preview and the running character.

IK targets also need migrating: legacy `SetIk` calls do not drive AG2. Use the new target parameters and the coordinate space expected by their consuming nodes. A world-space position, a model-space transform, and a direction are not interchangeable.

Do not assume complete feature parity just because a parameter has the same name. In particular, the legacy procedural hit-reaction inputs (`hit`, `hit_bone`, `hit_offset`, `hit_direction`, and `hit_strength`) do not currently provide an equivalent AG2 effect.

Migrate one behavior at a time, with a graph small enough to understand before adding the next layer.
