---
title: "Runtime Graph Builder"
created: 2026-11-04
updated: 2026-11-04
---

# Runtime Graph Builder

`Sandbox.Animation.GraphBuilder` builds an AG2 graph in C#. It uses the same animation runtime as an authored `.ag`: clips, blends, states, layers, and procedural adjustments still work the same way.

Use it when the graph's contents come from gameplay: a runtime-selected animation, a configurable character, or a small reusable animation setup. Use the graph editor when seeing and tuning the whole graph is more useful than generating it.

You can mix both approaches. An authored character graph can accept a small runtime-built action through an [External Graph slot](/animation/animgraph/runtime-graph-builder/external-graphs.md).

## Build once, drive with parameters

There are four steps:

1. Get a builder and choose its skeleton.
2. Declare all control parameters.
3. Connect nodes to describe the final pose.
4. Call `Create` and assign the resulting graph.

```csharp
using Sandbox.Animation;

var builder = Graph.Builder;
builder.Skeleton = "models/citizen_human/animgraphs/human.skel";

var speed = builder.FloatParameter( "speed", 0, 300 );
var idle = builder.Clip(
    "models/citizen_human/animations/clips/avatarmenu_idle_01.anim"
);
var walk = builder.Clip(
    "models/citizen_human/animations/clips/walk_forward.anim"
);

var movement = builder.BlendSpace( speed )
    .Add( 0, idle )
    .Add( 100, walk );

Graph graph = builder.Create( movement, "runtime_movement" );
```

The numbers are positions on the blend axis, not playback speeds. At `speed = 50`, this graph blends halfway between idle and walk. The model and clips must use the same skeleton resource.

Assign the graph to `Renderer.Animation.Graph`, keep Use Anim Graph enabled, then supply `speed` through `Renderer.Animation.Parameters.Set`. **Do not rebuild the graph every frame.** Nodes describe calculations; parameters supply their changing inputs.

## A complete looping-clip component

Add this component to your project. Assign its Renderer and an Animation Clip bound to the renderer's skeleton.

```csharp
using System;
using Sandbox;
using Sandbox.Animation;

public sealed class RuntimeClipExample : Component
{
    [Property] public SkinnedModelRenderer Renderer { get; set; }
    [Property] public Clip AnimationClip { get; set; }

    private Graph graph;

    protected override void OnEnabled()
    {
        if ( !Renderer.IsValid() )
            throw new InvalidOperationException( "Assign a skinned model renderer." );

        if ( AnimationClip is null || !AnimationClip.IsValid
            || AnimationClip.IsError || !AnimationClip.IsBound )
            throw new InvalidOperationException( "Assign a valid, skeleton-bound clip." );

        var builder = Graph.Builder;
        builder.Skeleton = AnimationClip.SkeletonName;

        var pose = builder.Clip( AnimationClip, new GraphBuilder.ClipSettings
        {
            Looping = true,
            SampleRootMotion = false
        } );

        graph = builder.Create( pose, "runtime_clip_example" );
        Renderer.UseAnimGraph = true;
        Renderer.Animation.Graph = graph;
    }

    protected override void OnDisabled()
    {
        if ( Renderer.IsValid() && Renderer.Animation.Graph == graph )
            Renderer.Animation.Graph = null;

        graph?.Dispose();
        graph = null;
    }
}
```

This component owns the renderer's graph assignment while enabled. Disabling it detaches its graph and restores the model's default graph, not a previously assigned override. Re-enable it after choosing another clip. Root-motion sampling is disabled for this in-place example.

## Ownership and lifetime

A graph is a **resource definition**, not one character's playback state. Assigning the same graph to multiple renderers gives each its own animation instance.

| How the graph is used | Who keeps it alive? |
|---|---|
| Assigned to `Renderer.Animation.Graph` | Your code must retain a strong managed reference, such as the component field above |
| Connected through `Animation.ExternalGraphs.Connect` | The slot retains the resource after a successful connection |
| Loaded from an `.ag` with `Graph.Load` | The resource system manages the shared asset |

For a main procedural graph, detach it from every renderer using it **before** calling `Dispose`. Do not put it in a `using` block that ends while the renderer still needs it.

For an external procedural graph, you can dispose your own handle after `Connect` succeeds. Disconnecting releases the slot's instance; animation recording can retain it longer. Disposing a disk-loaded graph does not unload the shared asset.

The optional `Create` name is a debugging label. It does not save an `.ag`, register an asset path, or make the result loadable with `Graph.Load`.

## Builder rules

**Keep one builder in a variable.** Every access to `Graph.Builder` creates a new builder. Nodes from different builders cannot be connected.

**Declare controls before anything else.** Even `Constant`, `ReferencePose`, and virtual-parameter calculations count as nodes. Adding a control parameter after a node throws.

**Inputs have types.** `GraphBuilder.Node` is an opaque connection, not a pose or number you can inspect in C#. A float input needs a Float node, a condition needs a Bool node, and `Create` needs a Pose node. The builder checks ownership and input kinds.

**Finish wiring before `Create`.** Changing a builder afterwards does not edit an already-created graph. Use parameters for live changes, or create and deliberately replace a graph when its structure changes.

**Create on the main thread.** Resource loading and `Create` require the main thread. Managed node descriptions can be assembled elsewhere, but do not load assets or create native resources there.

The fluent helpers for blends, selectors, layers, and state machines convert to `Node` automatically. A state machine also provides an explicit `Build()` method.

## Choose a topic

| I want to... | Read |
|---|---|
| Play, sample, blend, or select clips | [Clips and blends](/animation/animgraph/runtime-graph-builder/clips-and-blends.md) |
| Supply inputs and calculate values | [Values and parameters](/animation/animgraph/runtime-graph-builder/values-and-parameters.md) |
| Build actions and transitions | [State machines](/animation/animgraph/runtime-graph-builder/state-machines.md) |
| Add a masked action over movement | [Layers and masks](/animation/animgraph/runtime-graph-builder/layers-and-masks.md) |
| Aim, solve IK, or adjust root motion | [Procedural animation](/animation/animgraph/runtime-graph-builder/procedural-animation.md) |
| Make decisions from animation events | [Events and timing](/animation/animgraph/runtime-graph-builder/events-and-timing.md) |
| Play a runtime-selected clip or reuse another graph | [Referenced and external graphs](/animation/animgraph/runtime-graph-builder/external-graphs.md) |
