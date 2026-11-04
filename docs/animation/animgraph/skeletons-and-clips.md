---
title: "Skeletons and Clips"
created: 2026-11-04
updated: 2026-11-04
---

# Skeletons and Clips

AG2 keeps the model, skeleton, clips, and graph as separate assets.

| Asset | What it contains |
|---|---|
| `.vmdl` | The model you render |
| `.skel` | The bone hierarchy used by animation, plus authored bone masks |
| `.anim` | An animation clip authored for a particular skeleton |
| `.ag` | The rules for playing and combining those clips |

A graph's clips should use the **same `.skel` resource** as the graph. Two skeleton files with similar bone names are not interchangeable clip bindings.

## Create a skeleton

If your assets already include the correct `.skel`, use it. You do not need a new skeleton for every clip or graph.

Otherwise, choose **Skeleton** from the Asset Browser's **Animation** create category. You can also right-click an FBX, DMX, or SMD source file and choose **Create Skeleton**.

In the import dialog, select your skeleton source and set **Import Scale** to match your model. Leave **Root Bone** empty to import the whole hierarchy, unless you specifically need only part of it. You can assign a **Preview Model** to inspect the result.

Save the `.skel` in your project's `Assets` folder. Check the skeleton against your model before importing the rest of your animations.

## Import a clip

Choose **Animation Clip** from the **Animation** create category.

1. Choose an FBX, DMX, or SMD animation source.
2. Select the skeleton the animation will use.
3. Check the take and import scale. The full animation is imported unless you change the frame-range settings.
4. Click **Import...** and save an `.anim` in your project's `Assets` folder.

You can also import from the graph editor's **Animation Clips** panel. This uses the graph's selected skeleton for you.

The import dialogs copy source files from outside the project into `Assets`, so those sources stay with the project.

## Ordinary clips and additive clips

An ordinary clip describes a pose. An **additive** clip describes a change to another pose: for example, a small recoil movement added to a weapon hold.

Use an ordinary clip for your first graph. Do not connect an additive recoil clip directly to Graph Output and expect a complete character pose.

For additive animation, author the clip's subtraction/base settings and apply it through an **Additive** layer. Choosing Additive on a layer does not turn an ordinary animation into a correctly authored additive clip. See [layers and masks](layers-and-masks.md).

## Reusing old animation sources

You can reuse source animation files from an AG1 character. Import them as `.anim` assets against the new skeleton.

Preserve any important take, trimming, scale, playback-duration, and additive-base settings from the old setup. Importing the same FBX does not automatically reproduce every processing setting from ModelDoc.

ModelDoc sequence names and `.anim` paths are different things. An AG2 **Animation Clip** selects an `.anim` asset, not a sequence name embedded in the model.
