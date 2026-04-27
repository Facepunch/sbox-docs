---
title: "Learning Path: Game Developer"
icon: "🎓"
created: 2026-04-25
updated: 2026-04-25
---

# Learning Path: Game Developer

| Iteration | Goal | Time | What you can do at the end |
|---|---|---|---|
| 1. Make it work | Treat every concept as a black box; get a thing running | ~2 hours | A working scene with player movement, you've felt hotload |
| 2. Make it correct | Same concepts, real understanding; lifecycle, ownership, perf | ~half day | You can debug "why isn't this firing?" without guessing |
| 3. Make it shippable | Edge cases, multiplayer correctness, publishing | ~2 days | You can release a small game on sbox.game |

Read sequentially. Each iteration assumes you finished the previous one.

---

## Iteration 1: Make it work (~2 hours)

You'll get a moving cube on screen, save and hotload code, attach a player, push it around, and save the result. Every concept here you'll re-encounter in iteration 2 with the explanations.

The point of this iteration is **not** to understand. It's to feel the loop. Save → hotload → see effect → save again. Once that's in your hands, the explanations in iteration 2 land.

### 1.1 Install (5 min)
- [Download & Install](getting-started/installation.md) — Steam install. Open `sbox-dev.exe` once it finishes.

### 1.2 First project (15 min)
- [Your First Project](getting-started/first-project.md) — make a project, attach a `ModelRenderer` and a `SpinComponent`, hit Play.

After this you've seen: GameObjects, Components, `[Property]`, `OnUpdate`, hotload. **Don't worry about what they all are.** Just notice they exist.

### 1.3 First component (30 min)
- [Tutorials → Your First Component](tutorials/your-first-component.md) — line-by-line walkthrough of writing a `Component` from scratch. The retrieval prompts in this tutorial are doing real work — pause when they ask.

After this you've seen: the lifecycle methods (`OnAwake`, `OnStart`, `OnEnabled`, `OnDestroy`), `[Property]` variants like `[Range]`, sibling-component lookup with `Components.Get<T>()`. Still as black boxes.

### 1.4 Player movement (45 min)
- [Build a Third-Person Controller](tutorials/build-a-third-person-controller.md) — drop the built-in `PlayerController`, configure the camera, write a small companion script extending it.

After this you have a working playable scene. **Save it.** You'll come back to it in iteration 2 and improve it with proper understanding.

### 1.5 What you should feel by now
- The save → hotload → see effect loop is in your hands.
- You know `Component`, `GameObject`, `[Property]`, `OnUpdate` exist.
- You can attach things in the Inspector.
- You can read your own saved scene file (open `main.scene` in a text editor — it's JSON; you'll recognise the names).

You don't yet know *why* `OnUpdate` is different from `OnFixedUpdate`, what `[RequireComponent]` means, when to cache references, how `[Sync]` works, or what a `Prefab` is for. That's iteration 2.

---

## Iteration 2: Make it correct (~half day)

Same concepts, no longer black boxes. You'll go back to the scene from iteration 1 and improve it once you understand *why* the patterns are what they are.

### 2.1 Component lifecycle, properly (1 hour)
- [Component Lifecycle](scene/components/component-lifecycle.md) — the most important page on the wiki for game devs.
- [Component Methods Reference](scene/components/component-methods.md) — full enumeration of every overridable method.

This re-encounters `OnUpdate`, `OnAwake`, `OnStart`, `OnFixedUpdate` from iteration 1.4 — but now you'll know which to use when. Go back to your iteration 1 scene; if anything was in `OnUpdate` that should be in `OnStart`, fix it.

### 2.2 The Scene System, properly (1 hour)
- [Scene System](scene/index.md) — the model overall.
- [GameObject](scene/gameobject.md) — full API: tags, hierarchy, cloning, `IsValid()`.
- [Components Overview](scene/components/index.md) — `[RequireComponent]` vs `Components.Get<T>` vs `Components.GetOrCreate<T>` vs `Scene.GetAllComponents<T>` — when to use each.

This re-encounters GameObjects and Components from iteration 1. Now you know why `[RequireComponent]` is preferable to `Components.Get<T>()` for required dependencies, and why caching `Components.Get<T>()` in `OnStart` matters.

### 2.3 Code basics (1 hour)
- [Code](code/index.md) — what's distinctive about s&box C#.
- [Code Basics → Math Types](code/code-basics/math-types.md) — `Vector3`, `Rotation`, `Transform`, the quaternion gotchas.
- [Code Basics → Hotloading](code/code-basics/hotloading.md) — the practical pitfalls (default values, dictionary `Equals`, delegates).

This re-encounters hotload from iteration 1.1 — now you know why default field values don't change after hotload, what `[SkipHotload]` is for, and why your reflection caches need to be cleared on the hotload event.

### 2.4 Property attributes (30 min)
- [Property Attributes](editor/property-attributes.md) — every editor-facing attribute.

You've used `[Property]` and `[Range]` in iteration 1. Now you'll know about `[Group]`, `[ToggleGroup]`, `[ShowIf]`, `[HideIf]`, `[Title]`, `[FilePath]`, `[Advanced]`, `[KeyProperty]` — the rest of the inspector toolkit.

### 2.5 Prefabs (45 min)
- [Prefabs](scene/prefabs/index.md), [Instance Overrides](scene/prefabs/instance-overrides.md), [Prefab Templates](scene/prefabs/prefab-templates.md).

You haven't really used prefabs in iteration 1. Make one now: take your player from iteration 1.4, drag it into your assets folder to convert it to a prefab. Then [Spawn a Prefab](how-to/spawn-prefab.md) at runtime when the player presses a key.

### 2.6 Maps and scenes (30 min)
- [Maps overview](scene/maps/index.md), [Loading Maps](scene/maps/loading-maps.md).
- [Scene Mapping](scene/scene-mapping/index.md) — author your level inside the scene editor with the Mapping tool.

Build a small environment for your player to move around in. The scene file you saved in iteration 1.4 is now the foundation; expand it.

### 2.7 What you should feel by now
- You know which lifecycle method to override for a given task without guessing.
- You can read someone else's component and tell what it's doing without running it.
- You've made at least one prefab and instantiated it from code.
- You've encountered a hotload pitfall and know how to handle it.
- You can shape a scene with the Mapping tool, not just empty GameObjects.

You still don't know how to make any of it multiplayer, how to handle edge cases like network ownership, or how to publish. That's iteration 3.

---

## Iteration 3: Make it shippable (~2 days)

This iteration makes your iteration-2 scene survive contact with players. Multiplayer, edge cases, perf, publishing.

### 3.1 Networking, the model (1.5 hours)
- [Networking & Multiplayer](systems/networking-multiplayer/index.md) — the four pieces.
- [Networked Objects](systems/networking-multiplayer/networked-objects.md) — `NetworkSpawn`, `NetworkMode`.
- [Sync Properties](systems/networking-multiplayer/sync-properties.md) — `[Sync]`, `[Change]`, `SyncFlags`, `NetList`/`NetDictionary`.
- [Ownership](systems/networking-multiplayer/ownership.md) — `IsProxy`, `OwnerTransfer`, `Connection`.
- [RPC Messages](systems/networking-multiplayer/rpc-messages.md) — `[Rpc.Broadcast]`, `[Rpc.Host]`, `[Rpc.Owner]`.

This is the biggest jump in this learning path. You're now thinking about state running on multiple machines simultaneously.

### 3.2 Networking, applied (2 hours)
- [Tutorials → Build a Networked Lobby](tutorials/networking-basics.md) — full applied tutorial.
- [Network Helper](systems/networking-multiplayer/network-helper.md) — drop-in `NetworkHelper` for fast prototyping.

Take your iteration-2 scene and convert your player to be networked. Add a synced health value with `[Sync]` and a damage RPC with `[Rpc.Broadcast]`. Test with two local instances.

### 3.3 Networking, edge cases (1 hour)
- [Map Networking](scene/maps/map-networking.md) — how `MapInstance` interacts with snapshots.
- [Network Visibility](systems/networking-multiplayer/network-visibility.md) — bandwidth culling.
- [Custom Snapshot Data](systems/networking-multiplayer/custom-snapshot-data.md) — when standard `[Sync]` isn't enough.

This re-encounters `[Sync]` from 3.1 with the late-joiner / snapshot edge cases. By now you should be developing intuition for "is this state or an event?" — state goes into `[Sync]`, events into RPCs.

### 3.4 The sandbox boundary (45 min)
- [API Whitelist](code/code-basics/api-whitelist.md) — what's blocked, what's allowed.
- [Sandbox.Access](engine-internals/sandbox-access.md) — why the verification exists.

This re-encounters the API whitelist from iteration 2.3 — now you understand it's not just an annoyance but the security boundary that lets player code run on stranger machines safely.

### 3.5 Performance (1 hour)
- [Performance & Optimization](systems/performance/index.md), [Profiling & Overlays](systems/performance/profiling.md), [Optimization Guide](systems/performance/optimization-guide.md).
- [Use Async Tasks](how-to/component-async.md) — for work that shouldn't block the main thread.

Profile your scene. Find one allocation in `OnUpdate` or one repeated `Components.GetAll<T>()` and remove it. The pattern matters more than the specific fix.

### 3.6 Publishing (1 hour)
- [Publishing → Publishing Addons](publishing/publishing-addons.md) — uploading to sbox.game.
- [Publishing → Exporting Standalone](publishing/exporting-standalone.md) — for shipping outside s&box.
- [Monetization](getting-started/monetization.md) — Play Fund, revenue, expectations.

Push iteration 2's scene to sbox.game as a draft. Don't publish publicly yet; just go through the upload flow so you've felt it once.

### 3.7 What you should feel by now
- You can make a multiplayer game work, survive late-joiners, and not rubber-band.
- You can profile and pick the actual bottleneck, not the suspected one.
- You've published an addon (even as a draft) and seen the platform side.
- You can read someone else's networked component and tell which client owns what.
- You know what the API whitelist is for, not just that it exists.

---

## Reference layer

These don't fit in any iteration. You consult them when you have a problem.

- [Cookbook & How-To](how-to/index.md) — recipes for known tasks.
- [Component Reference](scene/components/reference/index.md) — every built-in component.
- [Engine Systems](systems/index.md) — physics, audio, UI, etc., by topic.
- [Glossary](glossary.md) — when you hit an unfamiliar word.
- [Troubleshooting](troubleshooting/index.md) — when something's broken.

## Specialised tracks (after iteration 3)

These weren't covered above and don't have learning paths yet. Pick them up when a project needs them:

- [Editor](editor/index.md) and editor extensions
- [ActionGraph](systems/actiongraph/index.md) (visual scripting)
- [Movie Maker](systems/movie-maker/index.md)
- [Shader Graph](systems/shader-graph/index.md) and [Shaders](systems/shaders/index.md)
- [VR](systems/vr/index.md)
- [Dedicated Servers](systems/networking-multiplayer/dedicated-servers/index.md)

