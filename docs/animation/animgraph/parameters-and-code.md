---
title: "Parameters and Code"
created: 2026-11-04
updated: 2026-11-04
---

# Parameters and Code

Keep gameplay decisions in game code and animation composition in the graph. Code supplies values such as speed and grounded state; the graph turns those values into poses.

## Control parameters

Add control parameters in the graph's **Parameters** panel, then drag them into the graphs that need them.

| Type | Example | Meaning |
|---|---|---|
| Float | `speed` | A continuous number |
| Bool | `b_grounded` | True or false |
| ID | `holdtype` | A named choice, such as `"rifle"` |
| Vector | `aim_target` | A direction or position, as required by the consuming node |
| Target | `hand_target` | An IK target, such as a transform or bone |

Names are a contract between your code and graph. Adding `speed` does not make it track movement automatically.

A parameter's **Runtime > Default Value** is its authored runtime starting value. **Preview > Start Value** is for the editor preview. Moving a preview slider is not the same as changing the runtime default.

## Drive the movement example

This small component lets you drive the [movement blend](blending-movement.md) from a property:

```csharp
using Sandbox;

public sealed class AnimationSpeedExample : Component
{
    [Property] public SkinnedModelRenderer Renderer { get; set; }
    [Property, Range( 0, 300 )] public float Speed { get; set; }

    protected override void OnUpdate()
    {
        Renderer.Animation.Parameters.Set( "speed", Speed );
    }
}
```

Assign Renderer, give that renderer your `.ag`, and leave Use Anim Graph enabled. The graph must declare a Float named `speed`. In play mode, change Speed on the runtime component to see the blend respond.

In your real movement code, supply horizontal velocity's length instead of this test property.

`Set` reports a missing parameter, incompatible value, or unavailable graph by throwing. Use `TrySet` when a parameter is deliberately optional, and handle its `false` result when that absence matters.

## Assign a graph from code

You can assign the graph in the inspector or load it from code:

```csharp
Renderer.Animation.Graph = Sandbox.Animation.Graph.Load( "graphs/character.ag" );
```

Use your own asset path, relative to `Assets`. Assign the graph when setting up or changing the character, not every frame.

`Renderer.Animation.Graph` returns the effective graph, including the model default. `Renderer.Animation.IsValid` tells you whether a runtime AG2 controller is available.

## Saved values and live values

Values edited in the renderer's animation parameter inspector are saved with the scene or prefab and reapplied when its animation instance is recreated.

Ordinary `Animation.Parameters.Set` calls change the live instance only. They are not saved with the scene and are not restored after a graph or model reload. Keep supplying continuously changing gameplay values.

Do not use persisted parameter writes as a substitute for runtime movement updates.

## One-shot requests

An action bool such as `b_wave` stays at its last value until you change it. Keep it true for an animation update, then clear it on the next one.

Do not set it true and immediately false in the same method: the graph may never see the true value. Also decide what should happen if another request arrives during the action; the [basic wave graph](states-and-transitions.md) ignores requests while already waving.

## Virtual parameters

A **control parameter** comes from outside the graph. A **virtual parameter** is a named calculation inside it.

For example, several branches might need the same smoothed movement speed:

```text
Inside virtual parameter smooth_speed:
speed ---> Spring ---> Result
```

Create a Float with **Add Virtual Parameter**, name it `smooth_speed`, and build that calculation in its value graph. Drag `smooth_speed` from Parameters wherever it is needed.

Leave **Lifetime** at **Graph** for a calculation whose initialized state should survive state changes. **Referenced Branches** keeps its value tree initialized only while referencing branches need it. When all those branches release it, a later activation starts it again; this matters for springs, random choices, and cached values.

Use virtual parameters to share derived values, not as extra gameplay inputs.

## Using PlayerController

The standard PlayerController supplies a set of optional inputs when its assigned renderer uses an AG2 graph. Your graph only needs to declare the ones it uses.

| Parameter | Type | Meaning |
|---|---|---|
| `move_groundspeed` | Float | Actual horizontal movement speed relative to the ground |
| `move_x`, `move_y`, `move_z` | Float | Forward, right, and up movement speeds |
| `wish_groundspeed` | Float | Desired horizontal movement speed |
| `b_grounded` | Bool | Grounded state; climbing also counts as grounded |
| `duck` | Float | `0` standing, `1` crouching, `2` low crouch |
| `b_jump` | Bool | A one-update jump request |

For the movement example, rename `speed` to `move_groundspeed` and remove the test driver. Let the controller supply the value instead of having two components write it.

This does not make an arbitrary AG2 graph understand the legacy CitizenAnimationHelper contract. See [moving from AG1](moving-from-ag1.md).
