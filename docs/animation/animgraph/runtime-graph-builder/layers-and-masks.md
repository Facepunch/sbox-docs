---
title: "Layers and Masks"
created: 2026-11-04
updated: 2026-11-04
---

# Layers and Masks

A layer adds animation over a base pose. Its weight controls how much it contributes; its mask controls which bones it affects.

Use an ordinary layer for a continuously weighted pose. Use a **state-machine layer** for an action that can turn off, start, and finish.

## Add an ordinary layer

After declaring all controls and building your poses:

```csharp
var result = builder.Layers( movement )
    .AddLayer( breathingPose, new GraphBuilder.LayerSettings
    {
        Weight = breathingWeight,
        BlendMode = GraphBuilder.LayerBlendMode.Additive
    } );
```

This example requires a breathing clip authored as an additive delta. An ordinary absolute animation does not become additive just because you choose an additive blend mode.

Layers apply in the order added. Each sees the result of the layers below it.

## Wave over movement

This fragment assumes `movement` is a base pose and `request` is an already-declared Bool. The wave clip and `upper_body` mask must belong to the builder's skeleton.

```csharp
var action = builder.StateMachine();
var off = action.AddOffState();
var wave = action.AddState( builder.Clip( waveClip,
    new GraphBuilder.ClipSettings
    {
        Looping = false,
        SampleRootMotion = false
    } ) );

var transition = new GraphBuilder.TransitionSettings { Duration = 0.2f };
action.AddTransition( off, wave, request, transition );
action.AddTransition(
    wave, off, action.Completed( wave, 0.2f ), transition
);

var result = builder.Layers( movement )
    .AddLayer( action, new GraphBuilder.LayerSettings
    {
        BoneMask = builder.BoneMask( "upper_body" ),
        BlendMode = GraphBuilder.LayerBlendMode.Overlay,
        Synchronized = false
    } );
```

Pass `action` itself to `AddLayer`, **not `action.Build()`**. The `StateMachineBuilder` overload marks this as a state-machine layer and carries the layer settings into its states.

A state-machine layer keeps evaluating while its weight is zero, so it can leave Off and finish transitions. Treating the same machine as an ordinary layer can prevent that behavior.

Off contributes no action pose. It does not play an idle over the torso. The base movement continues underneath, and the wave keeps its own timing. Clear the request as described in [State Machines](/animation/animgraph/runtime-graph-builder/state-machines.md).

## Layer settings

| `LayerSettings` | Purpose |
|---|---|
| `Weight` | Optional Float contribution weight |
| `BoneMask` | Optional BoneMask restricting the contribution |
| `BlendMode` | How to combine poses; defaults to `Overlay` |
| `Synchronized` | Synchronize timing to the base; defaults to `false` |
| `IgnoreEvents` | Exclude the layer's sampled events |
| `RootMotionWeight` | Optional Float root-motion contribution |

By default only the base contributes root motion. `IncludeLayerRootMotion()` enables layer contributions, and supplying a `RootMotionWeight` to `AddLayer` enables them automatically. Use `IncludeLayerRootMotion( false )` to restrict sampling to the base again.

For lower-level construction, `AddStateMachineLayer( node, settings )` adds an already-built machine node and returns its layer index. It does **not** transfer `Weight`, `BoneMask`, or `RootMotionWeight` into states. Supply those through `StateSettings` and explicitly enable layer root motion if needed. Prefer `AddLayer( stateMachineBuilder, settings )` for normal code.

## Blend modes

| `LayerBlendMode` | Use |
|---|---|
| `Overlay` | Blend an ordinary pose over the base |
| `Additive` | Add an authored local-space delta |
| `ModelSpace` | Blend model-space rotations with local positions/scales |
| `ModelSpaceRotationOnly` | Model-space rotation blending while preserving local positions and scale |
| `ModelSpaceAdditive` | Add model-space deltas using the base's original parent orientations |
| `ModelSpaceHierarchical` | Hierarchical model-space rotation blending |

Start with Overlay or Additive. Advanced model-space modes are for deliberately authored setups, not fixes for a wrong skeleton or incorrectly imported additive clip.

## Build and combine masks

Masks are authored on the `.skel`, not as old model weightlists.

| Method | Purpose |
|---|---|
| `BoneMask( "upper_body" )` | Use a named mask from the skeleton |
| `BoneMask( 0.5f )` | Uniform weight for every bone and float channel |
| `BoneMaskBlend( source, target, weight )` | Blend two masks; `0` uses source, `1` target |
| `BoneMaskSelector( id, defaultMask, options, settings )` | Choose from `(Id, Mask)` entries |
| `BoneMaskSwitch( condition, whenTrue, whenFalse, settings )` | Choose using a Bool |

Mask selection is entry-only unless you opt into dynamic switching:

```csharp
var mask = builder.BoneMaskSwitch(
    useFullBody,
    builder.BoneMask( 1.0f ),
    builder.BoneMask( "upper_body" ),
    new GraphBuilder.BoneMaskSelectionSettings
    {
        SwitchDynamically = true,
        BlendTime = 0.1f
    }
);
```

`BoneMaskSelectionSettings.SwitchDynamically` defaults to `false`, and `BlendTime` defaults to `0.1` seconds. Unlike pose selectors, mask selectors can explicitly opt into live changes.

`Scale( child, boneMask, enabled )` uses mask weights as **bone scale factors**. It does not mean "blend this pose by a mask." Its optional `enabled` input is a Bool. For masked animation composition, use a layer instead.
