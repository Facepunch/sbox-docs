---
title: "Referenced and External Graphs"
created: 2026-11-04
updated: 2026-11-05
---

# Referenced and External Graphs

An **External Graph** is an extension point in an animation graph. The main graph declares a named slot, and gameplay connects another graph to it at runtime. That child supplies a pose wherever the slot is used, without replacing the character's main graph.

The child can be much more than one clip: it can contain states, transitions, blends, layers, and procedural adjustments. It can be an authored `.ag` or a graph built in C#. The parent does not need to know which assets will be connected later.

For example, a game's main character graph can handle locomotion, while each equipped item supplies its own upper-body hold animation. Adding a fishing rod, camera, or new weapon then means creating another child graph, not adding another hold type to the main graph.

## What can you build?

| Use case | Parent graph owns | External child supplies |
|---|---|---|
| Custom item hold types | Locomotion, upper-body mask, live aiming and IK | An item's hold pose, idle loop, or draw-to-hold sequence |
| Runtime-selected actions | Where and how an action blends over the character | A reload, gesture, attack, or other selected animation |
| Optional cosmetic motion | An additive layer's mask and weight | Compatible, authored additive motion |
| Full-body sequences | When the sequence is allowed to take over | A self-contained interaction or cinematic sequence |
| Reusable animation extensions | Stable slot names and composition rules | Game- or item-specific graphs using that contract |

Slots are part of the parent's definition; `Connect` cannot add one to an arbitrary graph. They accept **graphs**, not models, components, or raw clips. Wrap a clip in a small graph to play it through a slot.

## External or internal?

A referenced graph, called **Internal Graph** in the editor, is a known child asset built into the graph definition. An external graph is chosen by gameplay at runtime.

| | Referenced graph | External graph |
|---|---|---|
| Builder method | `ReferencedGraph( path, fallback )` | `ExternalGraphSlot( name, fallback )` |
| Child chosen | When building the parent | When gameplay calls `Connect` |
| Parent parameter values | Compatible matching names are reflected into the child | Not inherited; child controls use their defaults |
| Typical use | Reusable authored locomotion driven by parent inputs | Item-specific behavior or a runtime-selected action |

Both produce poses and accept an optional fallback pose. Use the same skeleton resource throughout; matching bone names alone is not sufficient.

## How a slot behaves

An External Graph node outputs its connected child's result, or its **Fallback** when empty. It does not blend between them by itself.

Connecting creates a fresh child instance and shuts down the fallback if it was running. Disconnecting removes the child and reinitializes the fallback if the slot is initialized. The fallback does **not** continue invisibly underneath and resume where it left off.

Placement determines the effect:

```text
Locomotion ---> External Graph.Fallback
                External Graph.Result ---> Graph Output
```

This is a full-body replacement. While connected, the child replaces locomotion at the output. To keep walking underneath an item pose, put locomotion in **Layer Blend.Base** and the slot in a separate layer instead.

An empty slot without a fallback does not supply a useful pose. For an optional layer, give it an explicit fallback and make the layer's weight zero while empty. Do not expect an empty slot to behave like an Off State.

## Example: custom item hold types

Suppose the character already has a locomotion graph. You want each equipped item to choose how the arms hold it, while leaving the legs and movement logic alone.

The parent declares a slot called `hold_pose`. A rifle can connect `rifle_hold.ag`, a camera can connect `camera_hold.ag`, and a new item can provide another graph without changing the parent.

### Prepare the main graph in the editor

Use the [Layers and Masks](/animation/animgraph/layers-and-masks.md) guide to create an `upper_body` mask on the character's skeleton. Then wire the main graph like this:

```text
Locomotion --------------------------------> Layer Blend.Base

Reference Pose ---> External Graph.Fallback
                    Slot Name: hold_pose
                    Result ----------------> Local Layer.Input
Bone Mask (upper_body) ---------------------> Local Layer.Bone Mask
Is External Slot Filled ---> Float Switch -> Local Layer.Weight
                                             Local Layer.Layer ---> Layer Blend.Layer 0

Layer Blend.Result ---> Graph Output
```

Set **Slot Name** to `hold_pose` on both External Graph and Is External Slot Filled. Use exactly the same spelling and case.

Set Float Switch's **True Value** to `1`, **False Value** to `0`, and **Ease Time** to `0`. Set Local Layer's **Blend Mode** to **Overlay**, and leave **Is Synchronized** off. Leave Layer Blend's **Only Sample Base Root Motion** enabled.

With nothing equipped, the layer has zero weight, so the entire locomotion pose shows through. With an item connected, its graph supplies the masked upper body. The Reference Pose fallback is hidden by the zero weight; it is not an unarmed animation.

This deliberately starts with immediate switching. Simply increasing Float Switch's Ease Time does **not** implement a correct unequip fade: disconnecting would remove the item pose before it could fade out. See [smooth changes](#smooth-changes-and-interruption).

### The same parent setup in C#

This fragment assumes `builder` already has its skeleton, all control parameters are declared, and `movement` is a pose from that same builder:

```csharp
var hold = builder.ExternalGraphSlot( "hold_pose", builder.ReferencePose() );
var hasHold = builder.IsExternalGraphSlotFilled( hold );

var result = builder.Layers( movement )
    .AddLayer( hold, new GraphBuilder.LayerSettings
    {
        Weight = builder.Switch( hasHold, 1.0f, 0.0f ),
        BoneMask = builder.BoneMask( "upper_body" ),
        BlendMode = GraphBuilder.LayerBlendMode.Overlay,
        Synchronized = false
    } );

Graph graph = builder.Create( result, "character_with_item_hold" );
```

Assign and retain this main graph using the [builder ownership rules](/animation/animgraph/runtime-graph-builder/index.md#ownership-and-lifetime). Or use the editor-authored parent; gameplay connects children in exactly the same way.

### Author an item's child graph

Create a separate `.ag` using the parent's skeleton. For a simple hold type, connect a looping, non-additive **Animation Clip** directly to Graph Output. Disable the clip's root-motion sampling.

The child can output a full-skeleton pose; the parent applies the upper-body mask. It does not need another mask, an External Graph node, or its own copy of locomotion.

For a fixed grip rather than an animated idle, use **Animation Pose** to sample the desired frame from a pose sheet. For a more elaborate item, use a state machine with a non-looping Draw state followed by a looping Hold state. Make Draw the default and use **State Completed** to transition to Hold.

That sequence starts when the child is connected and evaluated. It needs no parent `b_draw` parameter. Connecting the same graph again after disconnecting creates a new instance, so the draw sequence starts again.

### Equip and unequip from gameplay

This component owns the `hold_pose` slot. Assign a renderer running the parent above, with Use Anim Graph enabled. Item code calls `Equip` with its graph and `Unequip` when the item is removed.

```csharp
using System;
using Sandbox;
using Sandbox.Animation;

public sealed class ItemHoldExample : Component
{
    [Property] public SkinnedModelRenderer Renderer { get; set; }

    private const string Slot = "hold_pose";
    private bool equipped;

    public void Equip( Graph holdGraph )
    {
        if ( !Renderer.IsValid() )
            throw new InvalidOperationException( "Assign a skinned model renderer." );

        if ( holdGraph is null || !holdGraph.IsValid || holdGraph.IsError )
            throw new ArgumentException( "Provide a valid hold graph.", nameof( holdGraph ) );

        var slots = Renderer.Animation.ExternalGraphs;
        if ( !slots.Contains( Slot ) )
            throw new InvalidOperationException( "The running graph needs a hold_pose slot." );

        if ( slots.IsConnected( Slot ) )
            slots.Disconnect( Slot );

        equipped = false;
        slots.Connect( Slot, holdGraph );
        equipped = true;
    }

    public void Unequip()
    {
        if ( equipped && Renderer.IsValid() )
        {
            var slots = Renderer.Animation.ExternalGraphs;
            if ( slots.Contains( Slot ) && slots.IsConnected( Slot ) )
                slots.Disconnect( Slot );
        }

        equipped = false;
    }

    protected override void OnDisabled()
    {
        Unequip();
    }
}
```

For an `ItemHoldExample` reference named `itemAnimator`:

```csharp
itemAnimator.Equip( Graph.Load( "graphs/items/rifle_hold.ag" ) );
```

Later, equip a different item or return to unarmed locomotion:

```csharp
itemAnimator.Equip( Graph.Load( "graphs/items/camera_hold.ag" ) );
itemAnimator.Unequip();
```

These are example asset paths relative to your project's `Assets` folder. Create your own graphs; there are no special built-in `rifle` or `camera` slot values. In real item code, store the graph on the item's resource or component rather than hard-coding every item in the animation driver.

Call `Equip` on an equipment change, not every update. Repeated calls restart the child. This example assumes the Renderer remains assigned to the same character; unequip before changing that reference. Controller recreation requires an explicit re-equip if the item should remain visible.

### Generate a draw-to-hold graph

You can also build the item's graph from clips selected by gameplay. This factory takes two valid, bound clips using the same skeleton as the parent:

```csharp
using Sandbox.Animation;

public static Graph CreateItemHoldGraph( Clip drawClip, Clip holdClip )
{
    var builder = Graph.Builder;
    builder.Skeleton = holdClip.SkeletonName;

    var machine = builder.StateMachine();
    var draw = machine.AddState( builder.Clip( drawClip,
        new GraphBuilder.ClipSettings
        {
            Looping = false,
            SampleRootMotion = false
        } ) );
    var hold = machine.AddState( builder.Clip( holdClip,
        new GraphBuilder.ClipSettings
        {
            Looping = true,
            SampleRootMotion = false
        } ) );

    machine.AddTransition(
        draw, hold, machine.Completed( draw, 0.15f ),
        new GraphBuilder.TransitionSettings { Duration = 0.15f }
    );

    return builder.Create( machine, "item_draw_and_hold" );
}
```

With compatible `drawClip` and `holdClip` values:

```csharp
using var holdGraph = CreateItemHoldGraph( drawClip, holdClip );
itemAnimator.Equip( holdGraph );
```

The successful connection retains the resource, so disposing this local procedural handle is safe. The child stays connected in Hold until unequipped. Build on equipment changes, not every frame.

### Keep live inputs in the parent

External graphs **do not inherit parent parameters**, even when their names match. `Renderer.Animation.Parameters.Set( "speed", value )` addresses the main graph, not a connected child. The external-slot API does not expose a child parameter setter.

For an item system, a useful division is:

| Put in the parent | Put in the item graph |
|---|---|
| Movement and grounded-state decisions | Fixed grip or looping item idle |
| Live aiming, hand IK, recoil controls | A self-contained draw-to-hold sequence |
| Layer masks, weights, and action priority | Clip selection and constants chosen when building the child |
| Gameplay-triggered action state/layer | Timing-driven transitions internal to an action |

Apply live aiming or IK to the composed pose in the parent where appropriate. A child can calculate values internally, but a control parameter such as `b_reload` will remain at its runtime default unless the graph's own design avoids needing external writes.

If a reusable child must continuously follow parent inputs, use an **Internal Graph** with compatible matching parameters instead. For a known set of parameter-driven hold types, authored parent states may be simpler. External slots are most useful when the supplied content is open-ended and can operate behind a small, explicit contract.

## Combine holds and actions

A single slot has one connected child. Use separate slots for effects that need to coexist. For example:

```text
Locomotion ---> Layer Blend.Base
hold_pose ----> masked hold layer ------> Layer Blend.Layer 0
action -------> masked action layer ----> Layer Blend.Layer 1
                                         Layer Blend.Result ---> aiming / IK ---> Graph Output
```

Build `action` with the same optional-slot setup as `hold_pose`: Reference Pose fallback, a separate Is External Slot Filled check, and an immediate `0`/`1` layer weight. Each layer has its own mask. Later layers apply over earlier ones, so an ordinary upper-body reload can override the hold pose while leaving the legs moving.

Keep the hold connected during the action. It continues to evaluate if its layer is evaluated, so the action does not require restarting the item's draw sequence. Disconnecting `action` reveals the current hold pose again. Use the one-shot player below for this `action` slot.

For additive recoil or other additive motion, use a separate deliberately additive layer and compatible additive content, not the ordinary Overlay action setup. A slot name does not impose a mask, blend mode, or priority; the parent does.

## Reuse a fixed authored graph

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

In the editor, the equivalent setup is an **External Graph** node with **Slot Name** set to `action`, locomotion connected to Fallback, and Result connected to Graph Output. The **Is External Slot Filled** node tests a slot by its Slot Name.

Keep publicly connectable slots in the main graph. The managed API does not traverse internal or external children to find nested slots, and it has no path syntax such as `item/action`.

## Play a runtime-selected one-shot

This is the AG2 replacement for the common AG1 Direct Playback use case: choose a clip in game code, play it once, and return to the parent graph.

The component below requires a renderer whose running graph declares an `action` slot. It works with the full-body slot above or the masked action layer in [Combine holds and actions](#combine-holds-and-actions). It owns that slot while playing. The clip must be valid, skeleton-bound, and compatible with the parent.

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

The example can use `NormalizedTime >= 1` because its child is a single **non-looping clip**. For a complex child graph, root timing is not a universal "all actions finished" signal; design an explicit completion contract instead. In particular, do not apply this rule to the draw-to-hold graph: reaching Hold means the item is ready, not that it should be disconnected.

Deactivating and later re-entering a branch can reinitialize its child playback. A connected graph is not a separate always-running animation player, and an inactive connection is not a promise to resume at the same frame.

A finished clip holds its last pose until disconnected. The graph does not remove itself or reset a gameplay bool.

## Connection API

| `Animation.ExternalGraphs` method | Purpose |
|---|---|
| `Contains( slot )` | Whether the running graph declares the slot |
| `IsConnected( slot )` | Whether a graph is connected, even on an inactive branch |
| `Connect( slot, graph )` | Start an independent child instance in an empty slot |
| `Disconnect( slot )` | Remove the child and restore fallback; an empty valid slot is unchanged |
| `TryGetPlaybackInfo( slot, out info )` | Read evaluated timing; returns `false` when unavailable/not connected |

The slot lookup only checks the main graph. Slot names must be nonblank. Connect/disconnect require the main thread.

`Connect` throws for an unavailable controller or slot, an occupied slot, an invalid/incompatible graph, or active recording playback. External children must use the parent's skeleton and cannot declare external pose slots. Disconnect before replacing or restarting a connection.

Connections are runtime-only. Graph/model reloads that recreate the controller lose them, and recording playback discards live connections. Reconnect deliberately when appropriate; do not silently restart an interrupted action every frame.

Each connection has its own playback state, even when several characters or slots share the same graph resource. Connecting does not transfer an existing character's state-machine progress. Slot operations are local runtime operations, not a networking protocol: synchronize the gameplay equipment/action decision through your game's networking and apply it to each relevant character instance.

While recording or retaining recorded data, disconnected instances and their resources can remain alive until the recording is cleared/replaced or the controller is destroyed.

## Smooth changes and interruption

Connecting and disconnecting do not add fades. For a small known action set, an authored [state-machine layer](/animation/animgraph/layers-and-masks.md) is usually simpler.

For runtime-selected actions or holds that need fades, separate **connection lifetime** from **visible weight**:

1. Connect the child while its parent layer is off or at zero weight.
2. Request the parent layer/state to blend in.
3. To stop, request blend-out but leave the child connected.
4. Disconnect only after the parent has finished fading it out.

For a state-machine layer, use Off and Active states, with the external slot in Active. Drive Off/Active transitions with a **parent** Bool, not solely Is External Slot Filled: the connection must remain filled during blend-out. Use **State Machine Layer** so the machine keeps processing transitions at zero layer weight. See [runtime layers](/animation/animgraph/runtime-graph-builder/layers-and-masks.md) for the builder overload.

For a single non-looping action, the parent can use State Completed to begin its return to Off early enough for the fade. Game code still owns disconnection; reaching Off does not empty the slot. Make the exit/cleanup signal part of your parent graph's contract, rather than guessing with a wall-clock delay while animation might not be evaluating.

Replacing an occupied slot is also an immediate change. To smoothly change rifle to camera, fade the old hold to zero, disconnect it, connect the new hold, then fade in. For a direct overlap crossfade, declare two slots with separately controlled layers/weights so both children can remain connected during the handover.

Choose interruption and ownership rules in gameplay: ignore a second action, replace it, or queue it. The API does not arbitrate competing components, queue actions, or provide a connection handle proving which component owns the current child.

## Define a slot contract

Treat slots as the public interface of your main graph. Document the expected skeleton, ordinary versus additive output, mask and root-motion policy, whether the child should loop, and who disconnects it. A slot named `hold_pose` does not enforce those rules.

For the examples here, `hold_pose` accepts an ordinary, in-place pose or loop on the character's skeleton and stays connected until unequipped. `action` accepts an ordinary, in-place one-shot and is disconnected by its player when evaluated playback completes. Both have one gameplay owner and use immediate layer switches.

Keep building and asset loading out of per-frame updates. Shared authored graphs are useful for many items or characters; build a procedural graph when its contents actually need to be selected/generated at runtime.

## Troubleshooting

| Problem | Check |
|---|---|
| `Contains` returns false | The renderer has a running AG2 graph and the slot is declared in that main graph, not only in a child |
| `Connect` throws | Slot spelling, an existing connection, graph validity, identical skeleton resource, external pose slots, or recording playback |
| Connecting a hold stops walking | The slot replaced Graph Output instead of feeding an upper-body layer over locomotion |
| The empty hold layer shows a reference pose | Its weight must be zero while disconnected |
| The hold follows the walk cycle's rhythm | Disable synchronization on the hold layer |
| Child `speed`, aim, or reload parameters never change | External children use their control defaults; keep live controls in the parent |
| A finished action stays visible | Non-looping clips hold their last pose until gameplay disconnects or blends them away |
| An unequip fade snaps or fades toward reference pose | The child was disconnected too early; keep it until the layer is off |
| Playback keeps restarting | Do not call Connect/Disconnect every frame; check branch re-entry too |
| Equipment disappears after a graph reload | Re-establish the connection after controller recreation if the gameplay item is still equipped |

## External Pose is different

`ExternalPose( slotName )` declares a pose supplied by native integration. Managed runtime pose assignment is not currently exposed. It is not the method for playing a `Clip`; use `ExternalGraphSlot` for that.
