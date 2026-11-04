---
title: "Referenced and External Graphs"
created: 2026-11-04
updated: 2026-11-04
---

# Referenced and External Graphs

A referenced graph is a known child asset built into the graph definition. An external graph is connected to a named slot by gameplay at runtime.

| | Referenced graph | External graph |
|---|---|---|
| Builder method | `ReferencedGraph( path, fallback )` | `ExternalGraphSlot( name, fallback )` |
| Child chosen | When building the parent | When gameplay calls `Connect` |
| Parent parameter values | Compatible matching names are reflected into the child | Not inherited; child controls use their defaults |
| Typical use | Reusable authored locomotion | Runtime-selected action or clip |

Both produce poses and accept an optional fallback pose. Use the same skeleton resource throughout.

## Reuse an authored graph

```csharp
var movement = builder.ReferencedGraph(
    "graphs/locomotion.ag",
    builder.ReferencePose()
);
```

Replace the example path with your own `.ag`. Each referenced node creates its own child instance. Use matching parent control/virtual parameter names and compatible types for values the child should receive.

This does not paste the child's nodes into your C# builder or let you mix node handles between builders.

## Add a runtime slot

```csharp
var action = builder.ExternalGraphSlot( "action", movement );
Graph graph = builder.Create( action, "character_with_action_slot" );
```

With nothing connected, the slot plays `movement`. Connecting an action replaces that output. This placement is full-body; place a slot inside a deliberately designed layer for partial-body actions.

Slot names must be nonblank and unique, case-insensitively. `IsExternalGraphSlotFilled( action )` returns a Bool node; its argument must be the node returned by `ExternalGraphSlot`.

In the editor, the equivalent setup is an **External Graph** node named `action`, with locomotion connected to Fallback and its result connected to Graph Output.

## Play a runtime-selected one-shot

This is the AG2 replacement for the common AG1 Direct Playback use case: choose a clip in game code, play it once, and return to the parent graph.

The component below requires a renderer whose running graph declares an `action` slot. It owns that slot while playing. The clip must be valid, skeleton-bound, and compatible with the parent.

```csharp
using System;
using Sandbox;
using Sandbox.Animation;

public sealed class RuntimeActionExample : Component
{
    [Property] public SkinnedModelRenderer Renderer { get; set; }

    private const string Slot = "action";
    private bool playing;

    public void Play( Clip clip )
    {
        if ( !Renderer.IsValid() )
            throw new InvalidOperationException( "Assign a skinned model renderer." );

        if ( clip is null || !clip.IsValid || clip.IsError || !clip.IsBound )
            throw new ArgumentException( "Provide a valid, skeleton-bound clip.", nameof( clip ) );

        var slots = Renderer.Animation.ExternalGraphs;
        if ( !slots.Contains( Slot ) )
            throw new InvalidOperationException( "The running graph needs an action slot." );

        var builder = Graph.Builder;
        builder.Skeleton = clip.SkeletonName;
        var pose = builder.Clip( clip, new GraphBuilder.ClipSettings
        {
            Looping = false,
            SampleRootMotion = false
        } );

        using var graph = builder.Create( pose, "runtime_action" );

        if ( slots.IsConnected( Slot ) )
            slots.Disconnect( Slot );

        slots.Connect( Slot, graph );
        playing = true;
    }

    protected override void OnUpdate()
    {
        if ( !playing )
            return;

        if ( !Renderer.IsValid()
            || !Renderer.Animation.ExternalGraphs.TryGetPlaybackInfo( Slot, out var info ) )
        {
            Log.Warning( "The action connection was lost; its animation controller may have been recreated." );
            playing = false;
            return;
        }

        if ( info.IsActive && info.NormalizedTime >= 1.0f )
            Stop();
    }

    public void Stop()
    {
        if ( playing && Renderer.IsValid() )
        {
            var slots = Renderer.Animation.ExternalGraphs;
            if ( slots.Contains( Slot ) && slots.IsConnected( Slot ) )
                slots.Disconnect( Slot );
        }

        playing = false;
    }

    protected override void OnDisabled()
    {
        Stop();
    }
}
```

Call `Play` with a loaded `Clip` from gameplay. Calling it again replaces/restarts the current action; `Stop` cancels it. For example, with a `RuntimeActionExample` reference named `actionPlayer`:

```csharp
actionPlayer.Play( Clip.Load(
    "models/citizen_human/animations/clips/avatarmenu_entry_wave_01.anim"
) );
```

This sample deliberately uses **immediate switches**, not crossfades. It needs no request bool: the connection starts playback, and explicit disconnection finishes it.

The local `using` is safe here because a successful external connection retains the graph resource. It would **not** be safe for a procedural graph assigned as the renderer's main graph without another strong reference.

## Completion means evaluated playback

`TryGetPlaybackInfo` reports the slot's last evaluated pose:

| `PlaybackInfo` | Meaning |
|---|---|
| `Duration` | Evaluated pose duration in seconds |
| `NormalizedTime` | Playback position from `0` to `1`; looping clips wrap |
| `IsActive` | Whether the slot contributed to the last graph update |

A connected slot can be inactive because its branch is not currently used. It does not keep playing independently in the background. Do not treat `IsConnected` as "currently animating," or use a wall-clock delay as a substitute for animation progress.

The example can use `NormalizedTime >= 1` because its child is a single **non-looping clip**. For a complex child graph, root timing is not a universal "all actions finished" signal; design an explicit completion contract instead.

A finished clip holds its last pose until disconnected. The graph does not remove itself or reset a gameplay bool.

## Connection API

| `Animation.ExternalGraphs` method | Purpose |
|---|---|
| `Contains( slot )` | Whether the running graph declares the slot |
| `IsConnected( slot )` | Whether a graph is connected, even on an inactive branch |
| `Connect( slot, graph )` | Start an independent child instance in an empty slot |
| `Disconnect( slot )` | Remove the child and restore fallback; an empty valid slot is unchanged |
| `TryGetPlaybackInfo( slot, out info )` | Read evaluated timing; returns `false` when unavailable/not connected |

The slot lookup does not search inside referenced child graphs. Connect/disconnect require the main thread.

`Connect` throws for an unavailable controller or slot, an occupied slot, an invalid/incompatible graph, or active recording playback. External children must use the parent's skeleton and cannot declare external pose slots. Disconnect before replacing or restarting a connection.

Connections are runtime-only. Graph/model reloads that recreate the controller lose them, and recording playback discards live connections. Reconnect deliberately when appropriate; do not silently restart an interrupted action every frame.

While recording or retaining recorded data, disconnected instances and their resources can remain alive until the recording is cleared/replaced or the controller is destroyed.

## Blending actions

Connecting and disconnecting do not add fades. For a small known action set, an authored [state-machine layer](/animation/animgraph/layers-and-masks.md) is usually simpler.

For runtime-selected actions that need fades, design the parent state/layer to blend the slot's contribution. Keep the external graph connected until the blend-out has finished; disconnecting at its start removes the pose you are trying to fade out.

External child parameters are not writable through this slot API and do not inherit parent values. Build a self-contained child with suitable defaults, or put live controls and composition in the parent graph.

## External Pose is different

`ExternalPose( slotName )` declares a pose supplied by native integration. Managed runtime pose assignment is not currently exposed. It is not the method for playing a `Clip`; use `ExternalGraphSlot` for that.
