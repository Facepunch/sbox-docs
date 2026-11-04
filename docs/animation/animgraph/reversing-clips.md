---
title: "Reversing Clips"
created: 2026-11-04
updated: 2026-11-04
---

# Reversing Clips

You can play clips backward in a graph or sample their poses in reverse order. Choose the approach based on whether you need automatic playback, events, root motion, or direct control over the sample time.

| What you need | Use |
|---|---|
| Play an existing clip backward | **Animation Clip > Play In Reverse**, on an unsynchronized branch |
| Choose forward or backward from gameplay | A Bool parameter connected to **Play In Reverse** |
| Control the exact playback position in either direction | **Time Controlled Animation Clip**, with matching Time and Play In Reverse inputs |
| Sample poses backward without events or root motion | **Animation Pose**, with decreasing Time |
| A reversed asset that plays normally in synchronized blends | Reverse/bake the animation in your source animation tool, then import it as a separate `.anim` |

## Reverse an Animation Clip

Start with an Animation Clip connected directly to Graph Output:

1. Add a **Bool** constant and set it to true.
2. Connect it to the clip's **Play In Reverse** input.
3. Keep **Speed Multiplier** positive, such as `1`.
4. Disable **Loop** if it should play backward once.

```text
Bool (true) ---> Animation Clip.Play In Reverse
                Animation Clip.Result ---> Graph Output
```

Set the reverse input before the clip starts to play from the end toward the beginning. For a gameplay-controlled direction, connect a Bool control parameter instead of the constant.

Changing that bool during playback changes direction around the current pose; it is not a general "restart at the end" command. Re-enter the state or use an explicit **Reset Time** request when you want a fresh playback.

A non-looping reversed clip holds its first source pose when finished. It still needs a [state completion transition](/animation/animgraph/states-and-transitions.md) or [external-graph disconnect](/animation/animgraph/runtime-graph-builder/external-graphs.md) to stop contributing.

## When Play In Reverse is ignored

A normal Animation Clip does not read its reverse input during a **synchronized update**. Its parent supplies the time instead.

This includes clips driven by:

| Parent setup | Why reversal does not work independently |
|---|---|
| **Blend 1D**, **Blend 2D**, or **Blend 1D (Velocity)** | The blend synchronizes its source clips |
| A **Local Layer** or **State Machine Layer** with **Is Synchronized** enabled | The layer follows the base pose's timing |
| A synchronized state-machine transition | The transition supplies synchronized timing while active |

The C# builder has the same restriction. `Blend`, `BlendSpace`, `BlendSpace2D`, and `VelocityBlend` synchronize their inputs. In particular, `Blend` is a two-point blend space, not an unsynchronized crossfade.

The graph compiler warns about reverse inputs under synchronized blends or layers. Check **Compile Log** even if the graph still compiles.

For an independent reversed action, use an unsynchronized state-machine branch or layer. **Keep the path inside that branch unsynchronized too:** disabling synchronization on the outer layer does not make a Blend 1D inside it stop synchronizing its clips.

If the animation must participate in a synchronized locomotion blend, import an already-reversed source animation and play that clip forward. If you deliberately want to own its clock instead, use a time-controlled clip—but it will no longer follow the parent's synchronization.

## Reverse a time-controlled clip

**Time Controlled Animation Clip** samples the source position supplied to **Time**. It ignores parent synchronization.

For backward playback, do both:

1. Drive **Time** from `1` down to `0`.
2. Set **Play In Reverse** to true so event and root-motion sampling interpret that movement as backward playback.

The reverse bool does **not** turn an increasing Time input into a decreasing source position. You still supply the desired position yourself. If you have a progress value increasing from `0` to `1`, use `1 - progress` for Time.

Conversely, decreasing Time while Play In Reverse is false is interpreted as wrapping forward through the loop for event and root-motion sampling. The pose may look right while events or movement are wrong.

Keep the supplied position within `0` to `1`. For arbitrary scrubbing where you do not want playback events or motion, use Animation Pose instead.

## Sample poses in reverse

**Animation Pose** has no playback direction or automatic clock. Drive its normalized **Time** from `1` to `0` to sample the poses in reverse order. With **Use Frames As Input**, drive from the last frame down to the first.

This samples poses only: it does not sample animation events or root motion. It is useful for a manually positioned door, lever, or pose sequence—not a drop-in replacement for an action that needs playback events.

## What does not reverse an animation

**Negative speed is not reverse playback.** A negative Speed Scale is treated as zero. Negative Duration Scale or desired velocity inputs are not reverse controls either. Keep the clip's Speed Multiplier positive and use the reverse input.

A single-frame clip has no sequence of poses to reverse. Also, there is no general-purpose node that reverses the completed output of a blend, layer stack, or state machine: choose direction at the clips, drive sample time yourself, or prepare a reversed asset.

Reversing poses does not undo gameplay. Review event meanings, root motion, and any warp markers before reusing an action backward. When importing an already-reversed source, review its `.anim` event timings as well.

For C# examples, see [reversing clips with the runtime builder](/animation/animgraph/runtime-graph-builder/clips-and-blends.md#reverse-a-clip).
