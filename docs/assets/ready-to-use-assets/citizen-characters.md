---
title: "Human Characters"
icon: "🧍"
created: 2024-07-29
updated: 2026-11-15
---

# Human Characters

The Human character is the default player model provided by Facepunch. Use `models/citizen_human/human.vmdl` in your game.

# Character source files

**All source files come with the game: VMDLs, FBXs, animgraphs, etc.**

You can find them right in the game folder, under `addons/citizen/Assets/models/citizen_human`. Start with `human.vmdl` to inspect the model, and `citizen_human_male_REF.fbx` or `citizen_human_female_REF.fbx` when making clothing. These files are synced straight from our source folders; when new source files are added on our end, they'll show up in that folder for you too!


:::info
The asset folders and reference filenames still contain `citizen`. These are the current paths for the Human assets. The old Citizen player model is retired as the default and is no longer available in avatar selection, but its asset remains available to existing games.

:::

## Procedural helper bones & constraints

These helper bones are set up to be procedural helpers driven entirely by constraints configured in ModelDoc. Open up `human.vmdl` and look at the *AnimConstraintList* prefab, `models/citizen_human/prefabs/human_animconstraintlist.vmdl_prefab`, to see what makes them tick!

* `arm_upper_[R/L]_twist[0/1]` (from shoulder to biceps)
* `arm_elbow_helper_[R/L]` (better deformation for extreme bend angles on the elbow)
* `arm_lower_[R/L]_twist[0/1]` (from forearm to wrist)
* `leg_upper_[R/L]_twist[0/1]` (from pelvis to thigh)
* `leg_knee_helper_[R/L]` (better deformation for extreme bend angles on the kneecap)
* `leg_lower_[R/L]_twist[0/1]` (from calf to foot)
* `neck_clothing` (reduces the twist of the neck by two-thirds; some clothing items use it)

Because they're procedural, animation data doesn't need to be exported from your 3D program for these; in fact, it's set up to be ignored. Our own animation FBX files usually don't have them exported, which has the added benefit of slightly saving on file size. 

If you're making a model (e.g. clothing) to be bonemerged on top of the Human, use the supplied reference mesh's skinning as a guide for these helper bones.


:::warning
The limb bone is a sort of "container" for its `*_twist{0/1}` bones. **The limb bones aren't skinning anything and should never skin anything**; the twist bones are. Think of them as the "upper part" and the "lower part" of each limb respectively. This is how the height scaling feature works.

:::

# Adding new animations

If you only need to add a few animations to the VMDL, you can try using the "Base Model" feature. This is similar to the Source 1 "$includemodel" feature. **You can only add animations to an existing model using this feature; you can't add anything else.** Reference `models/citizen_human/human.vmdl` in the "Base Model" field, above the "Add" button in ModelDoc.


:::warning
Name your new animations something unique (maybe by giving them all a prefix); bad things could happen if animation names collide.

:::

You could also make your own copy of `human.vmdl` in your project while keeping the references to prefabs intact, but you might have to sync some changes manually.

## IK bones

These bones are used to drive IK constraints in-engine. **They need animation data, and the default graph assumes it is present.** This data is baked by the 3D animation program during the exporting process.


`root_IK` is the parent to all `*_IK_target` bones. These are effectively "model-space" IK targets.

### **Why are we doing it this way?**

Because the position of IK targets need to be kept in a different space that won't be affected by any layers, weightlists, etc. that animation compositing is touching, otherwise we can't restore the positions through IK afterwards! Think of them as a way to keep the pos/rot data of hands and feet intact, separately.

`root_IK` is, in the control rig, constrained to follow X & Y of the pelvis, but with Z (height) forced to 0. It also needs to always point forwards (zero rotation), otherwise funky things can happen. Similarly, the hand/feet target bones are pos/rot constrained to their respective hand/feet bones in the control rig.

Additionally, there are two special IK bones, which are used to reproduce the `ikrule touch` feature from Source 1.

* `hand_L_to_R_ikrule` (child of right hand, constrained to the left hand)
* `hand_R_to_L_ikrule` (child of left hand, constrained to the right hand)

These are used as a way to keep the position and orientation of hands the same *relative to each other* (in their local space), even after applying layers, weightlists etc. and they are pos/rot constrained in the control rig.

In the animgraph, this is most often how the left hand is made to stick to a two-handed weapon that's driven by the right hand! It's not sticking to the gun itself, it's made to stick back to its original position relative to the right hand, even after all the aim matrices, breathing additive motion, etc. has been applied.

There may be different IK solutions implemented in the future.

## LOD reference & guidelines

Inspect the Human model's *LODGroupList* in ModelDoc for its current meshes and switch thresholds. The retired Citizen's triangle counts and LOD distances do not describe the Human model.

Clothing should reduce detail as the character gets farther from the camera. s&box makes bonemerged models follow the LOD level of their parent, so check your clothing together with the Human model. See [Guidelines & Quality Bar](/assets/clothing/guidelines-quality-bar.md) for clothing LOD guidance.


:::tip
Here's a decent rule of thumb for telling at which distances your model needs to switch down a LOD level. Zoom out slowly, and look at when the wireframe starts looking like "solid green". Of course, it doesn't tell the whole story, but it's a good place to start from.

:::

If you are using more than one material, and can cull the number of total materials back to one in your LOD meshes, you should do so!

## Stretching limbs

Animations can change a bone's position in its parent's local space, changing the apparent length of a limb without storing a scale value. For example, moving a lower arm's local position from X = 10 to X = 12 stretches the upper arm by 20%.

The Human rig includes elbow and knee helpers to support deformation. Test custom animations and clothing on the Human mesh, especially when changing limb lengths or using extreme poses.

## About source file scaling

All source files are in centimeters. But because just about everything else is using inches, the model is scaled down using the **ScaleAndMirror** modifier in ModelDoc. (We're NOT doing it at the import level on each mesh node.)

Therefore, if you want to create something for the character (clothing, etc.), you should also work in centimeters, with the provided sources, and in your VMDL, you'll apply the same ScaleAndMirror modifier with the same value of **0.3937**. (4 decimals only is an arbitrary choice on our part.)

### Why?

This way, no one has to do any scaling using 3D packages, which are notoriously unreliable and inconsistent in how they choose to perform the scaling, especially during export steps. Also, the ability to flawlessly and arbitrarily scale any 3D model, even animated ones, including all their data, is extremely valuable to have, and important enough to be dogfooding.
