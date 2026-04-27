---
title: "Skeletal Animation"
icon: "🦴"
created: 2025-11-17
updated: 2025-11-26
---

# Skeletal Animation

You can animate skinned characters directly in the editor using Movie Maker. This tutorial covers:

* Enabling the skeletal animation tools
* Importing animation sequences from a model
* Generating bone animation track data using AnimGraph
* Recording bone animations from gameplay
* Upgrading legacy procedural bone object animations
* Using IK to manipulate poses

# Getting Started

Movie Maker's skeletal animation tools are enabled if you have at least one *SkinnedModelRenderer* in your movie with a *Bones* sub-track. When enabled for a renderer, you'll see its skeleton.

You'll also need to be in the [Motion Editor](motion-editing.md) to do the modifications described in this tutorial.

![Skeleton visualization setup](./images/skeleton-visualization-setup.mp4)

# Base Animations

Animating skinned objects from scratch can be time consuming, so you'll usually want to start with an imported or generated base-level animation that you can tweak later.

## Importing Animation Sequences

For simple sequences you can just load an animation from your model.

1. Make sure you're in *Motion Editor* mode
2. In the timeline, select a time range that you want to import an animation for
3. Right-click on a selected region of track that belongs to the object you want to animate
4. Select *Import Anim Sequence*, and pick a clip
5. Drag the imported sequence around or trim the start / end times
6. Click the green *Apply* button to save the modification

![Importing animation sequences](./images/importing-animation-sequences.mp4)

:::tip
Root motion will be included by default, you can disable it by locking the *LocalPosition* / *LocalRotation* tracks of the object you're animating.

:::

## Generating Using AnimGraph

You can save a lot of time by letting AnimGraph generate an animation for you. You'll just need *LocalPosition* and *LocalRotation* tracks describing how your character moves. These can be created in any way you like: keyframes, recording, or motion editing.

### Rotate With Motion

If you've only got a *LocalPosition* track, you can generate a *LocalRotation* track that always looks in the direction of movement. Just select the time range you want to generate rotations for in the motion editor, then right-click and select *Rotate with Motion* in the context menu, and apply if you're happy with it.

![Generating rotation from motion](./images/generating-rotation-from-motion.mp4)

Before moving on to the next step, feel free to tweak the generated *LocalRotation* track if you want your character to side-step or walk backwards at any point.

### Motion to AnimGraph Parameters

When you have *LocalPosition* and *LocalRotation* tracks for your *SkinnedModelRenderer*, you can generate AnimGraph parameter tracks based on that motion. Select the time range you want to generate the track data for, and select *Motion to AnimGraph Parameters* in the context menu.

![Converting motion to parameters](./images/converting-motion-to-parameters.mp4)

You can find the generated tracks nested under *Parameters* in the track list, and again they can be tweaked using keyframes or motion editing before moving to the next step.

### AnimGraph Parameters to Bones

If you want to have more fine-grained control over how your character moves, you'll need to bake its animation into bone tracks. As always, select the time range you want to bake, then select *AnimGraph Parameters to Bones* in the context menu. Now you're ready to move on to the [Manipulating Poses](#h-manipulating-poses) section.

![Baking animation to bones](./images/baking-animation-to-bones.mp4)

## Recording Gameplay

Another way to generate initial bone track data is to record it in play mode. You'll just need to create tracks for the bones you want to record before you start. We've included a sub-track preset for the built-in player controller as an example.

![Recording bone animation](./images/recording-bone-animation.mp4)

## Upgrading Legacy Bone Object Animations

Before we had dedicated bone track support, the best you could do was to enable *Create Bone Objects* on a *SkinnedModelRenderer* and add all the created objects to your movie. If you have an existing animation using that method, you can upgrade it to the new-style bone tracks with this context menu option. After doing the upgrade, you can manually delete the old tracks and disable *Create Bone Objects*.

![Upgrading legacy animations](./images/upgrading-legacy-animations.mp4)

# Manipulating Poses

We have some basic tools to help you tweak how your characters are posed during your movie. This is fairly minimal currently, but should be enough to do things like layer upper body motion on top of a walk animation generated using the above methods.

:::tip
You should know the basics of [Motion Editing](motion-editing.md) before getting started here.

:::

## Moving Bones

First, select a time region in your movie that you want to animate a pose change for. Next, click and drag from a joint. We'll simulate an IK chain going from the dragged bone to the root to try to find a sensible pose. While dragging, you can press *Esc* to cancel. Press the green *Apply* button when you're happy to commit the change to your animation.

![Moving bones with IK](./images/moving-bones-with-ik.mp4)

## Rotating Bones

When dragging a bone, you can hold the E key to start rotating it in place.

![Rotating bones in place](./images/rotating-bones-in-place.mp4)

## Locking Bones

Bones can be locked or unlocked in the context menu, or by holding *Shift* and clicking on them. This can be especially useful for things like finger posing by locking the hand.

![Locking bones for posing](./images/locking-bones-for-posing.mp4)

:::warning
Currently this will only animate correctly if you lock an ancestor of whichever bone you're manipulating. You can lock descendant bones like in the example below, but the lock is only considered for the current playhead time while posing.

:::

![Locking descendant bones](./images/locking-descendant-bones.mp4)
