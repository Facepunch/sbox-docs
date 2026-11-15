---
title: "AnimGraph"
created: 2026-11-04
updated: 2026-11-15
---

# AnimGraph

An animation graph turns gameplay values into a character's pose. Your game says "I'm moving at this speed" or "start waving"; the graph decides which clips to play and how to blend them.

**AnimGraph (AG2)** uses `.ag` files. This guide covers the new system, not the legacy `.vanmgrph` editor.

## How a graph works

The simplest graph plays one animation:

```text
Animation Clip (idle) ---> Graph Output
```

An **Animation Clip** produces a pose: the positions and rotations of the character's bones at that moment. **Graph Output** is the pose you want the character to use.

Other nodes can blend poses, choose between them, or adjust individual bones before the result reaches the output. **Parameters** carry values such as speed, grounded state, or an aiming target into these decisions.

There are three kinds of connections to recognize:

| Connection | What it carries | Example |
|---|---|---|
| Pose | An animation result | A walk clip feeding a blend |
| Layer | A pose with layer settings | Local Layer feeding Layer Blend |
| Value | A number, bool, ID, vector, target, or mask | A speed parameter controlling that blend |

Unlike ActionGraph, pose connections are not execution signals. You are describing how to produce a pose, not a sequence of commands. A clip in an inactive state does not keep playing in the background independently.

## Sharing poses and values

**Each pose or layer output connects to one input in the editor.** Connecting it to another input replaces its previous connection. Reroutes and Pose Groups do not bypass this rule.

Value outputs can connect to multiple inputs. For example, the same speed parameter can drive several blends, and the same bone mask can control several layers.

If two branches need the same animation, duplicate the clip node or the pose branch, not the animation asset. Both clip nodes can reference the same `.anim`, but each has its own playback state. Sharing a pose node is not a way to evaluate it once and reuse a cached result.

Older graphs with shared pose or layer outputs keep their saved connections, but compilation reports warnings on the affected nodes. Replace the sharing with separate branches. For branches that should follow the base animation's timing, use [layer synchronization](/animation/animgraph/layers-and-masks.md#keep-pose-branches-separate).

## Which node should I use?

| I want to... | Start with |
|---|---|
| Play an animation | **Animation Clip** |
| Blend idle, walk, and run from speed | **Blend 1D** |
| Blend movement from two values, such as forward and sideways speed | **Blend 2D** |
| Change behavior while playing, such as idle to wave | **State Machine** |
| Pick a variation once when entering a branch | **Selector** |
| Add a pose over another pose | **Local Layer** and **Layer Blend** |
| Play an action layer with its own states | **State Machine Layer** and **Layer Blend** |
| Restrict a layer to part of the body | **Bone Mask** |
| Share one graph structure across character-specific animation setups | **Variations** |
| Reuse an authored graph driven by parent parameters | **Internal Graph** |
| Let items supply custom holds or actions at runtime | **External Graph** |

**A Selector chooses on entry, not continuously.** If changing a parameter should switch the animation immediately, use a state machine or a blend instead.

For the full list, see [Node Reference](/animation/animgraph/node-reference.md): editor node names, what each one does, and the important restrictions.

## Learn by building

Start with [your first graph](/animation/animgraph/getting-started.md), then extend it:

1. [Blend movement](/animation/animgraph/blending-movement.md) from one speed value.
2. [Add states and transitions](/animation/animgraph/states-and-transitions.md) to play a wave and return.
3. [Use layers and masks](/animation/animgraph/layers-and-masks.md) to wave while the legs keep moving.

[Skeletons and clips](/animation/animgraph/skeletons-and-clips.md) explains asset preparation. [Parameters and code](/animation/animgraph/parameters-and-code.md) connects the graph to your game.

[Animation Events](/animation/animgraph/events.md) explains typed callbacks, ID markers, foot phases, native events, and graph events, with clip-authoring steps and a C# receiver.

[Variations](/animation/animgraph/variations.md) let one graph serve a family of characters or items, with inherited animation data and targeted overrides.

[Internal Graphs](/animation/animgraph/internal-graphs.md) explains reusable child assets, parameter passing, and shared locomotion. [External Graphs](/animation/animgraph/external-graphs.md) shows how to extend a main character graph with custom item hold types, draw-to-hold sequences, and runtime-selected actions, using either the editor or C#.

Want to construct graphs in C#? [Runtime Graph Builder](/animation/animgraph/runtime-graph-builder/index.md) covers clips, blends, states, layers, procedural animation, and runtime-selected actions.

Coming from the old editor? [Moving from AG1](/animation/animgraph/moving-from-ag1.md) maps the old workflow to the new one.
