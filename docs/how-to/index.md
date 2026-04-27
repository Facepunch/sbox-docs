---
title: Cookbook & How-To Guides
icon: "🧾"
updated: 2026-04-25
created: 2026-04-27
---

# Cookbook & How-To Guides

How-to guides are *problem-oriented* — recipes for known tasks, assuming you already know the basics. If you're learning rather than solving, use [Tutorials](../tutorials/index.md).

Recipes. Each page answers a specific "how do I X" question with the minimum code to get it working. Pick the one that matches what you're trying to do, copy, adapt.

These assume you already know what a Component is and how to attach one. If you don't yet, do [Your First Project](../getting-started/first-project.md) first.

## Index

### Components & GameObjects
- [Create a Component](create-a-component.md)
- [Manage Components](manage-components.md)
- [Query Components](component-queries.md)
- [Find GameObjects](find-gameobjects.md)
- [Clone GameObjects](clone-gameobjects.md)
- [Spawn a Prefab](spawn-prefab.md)
- [Create an Object Spawner](object-spawner.md)

### Movement & Physics
- [Player Movement](player-movement.md)
- [Apply Physics Forces](apply-physics-forces.md)
- [Rotate an Object](rotate-object.md)
- [Move an Object Smoothly](move-object-smoothly.md)
- [Camera Follow Player](camera-follow.md)
- [Trace / Raycast](trace-raycast.md)
- [Create a NavMesh Obstacle](navmesh-obstacle.md)

### Combat & Gameplay
- [Deal and Take Damage](deal-damage.md)
- [Detect Player Input](detect-input.md)
- [Attach an Object to a Bone](attach-weapon.md)

### Animation
- [Animate a Model](animate-model.md)
- [Use Inverse Kinematics (IK)](use-ik.md)

### UI & Effects
- [Create a UI Health Bar](health-bar.md)
- [UI Drag and Drop](ui-drag-and-drop.md)
- [Spawn Particles](spawn-particles.md)
- [Play Sounds from Code](play-sounds.md)
- [Load Image from URL](load-image-from-url.md)

### Networking
- [Sync Variables](sync-variables.md)

### Assets & Pipeline
- [Import a Map from Blender](import-map-from-blender.md)

### Persistence & Loading
- [Save and Load Data](save-and-load.md)
- [Use Cookies](use-cookies.md)
- [Change Scenes](change-scenes.md)
- [Use Async Tasks](component-async.md)
- [Async Component Loading](async-loading.md)

## When a recipe doesn't work

Three things to check first, in order:

1. **`using Sandbox;` at the top of the file.** Most engine types live there.
2. **Your `[Property]` references are assigned in the Inspector.** A null reference at runtime usually means you didn't drag a target into the property in the Editor.
3. **No red errors in the editor console.** Hotload pauses on compile errors; nothing else will work until you fix them.

If a recipe assumes a component exists (`Components.Get<Rigidbody>()`) and your GameObject doesn't have one, the recipe will null-ref. Either add the component manually in the Inspector, mark the property `[RequireComponent]`, or use `Components.GetOrCreate<T>()`.

## Past the cookbook

When a recipe isn't enough — you need to understand *why* something works the way it does — go to the systems documentation:

- [Engine Systems](../systems/index.md) for deep dives by topic
- [Component Reference](../scene/components/reference/index.md) for every built-in component's properties
- [Scene System](../scene/index.md) for the GameObject/Component model itself
- [Architectural Overview](../architecture.md) for how the engine fits together

Or if you're debugging by symptom rather than topic, [Troubleshooting](../troubleshooting/index.md) is organized that way.
