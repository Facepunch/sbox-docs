---
title: "Scene System"
icon: "🪑"
created: 2023-11-14
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Scene/GameObject/GameObject.cs
  - engine/Sandbox.Engine/Scene/Components/Component.cs
---

# Scene System

A scene is a tree of GameObjects. Each GameObject is just a transform plus a list of Components. Behaviour, rendering, physics — anything you'd think of as "what this thing does" — comes from the components.

If you've used Unity, this is the same model. If you haven't: keep reading.

## A minimal example

```csharp
var go = new GameObject( true, "My Object" );

var renderer = go.Components.Create<ModelRenderer>();
renderer.Model = Model.Load( "models/dev/box.vmdl" );

go.WorldPosition = new Vector3( 0, 0, 10 );
```

You make a GameObject, attach a `ModelRenderer` to it, give that renderer a model, and place the object in the world. That's the loop you'll repeat thousands of times.

## What's actually in the tree

Every scene has a single root that owns top-level GameObjects, each of which can own children. Walking children is `foreach (var child in go.Children)`; setting a parent is `child.SetParent(parent)`. Transforms compose down the tree — moving a parent moves its children with it.

GameObjects don't *do* anything. Components do. The components you attach decide whether the object renders, collides, plays sound, listens for input, syncs over the network. If you want a building block that does X, your job is to find or write the component for X.

## Where to go from here

The four core reads, in order:

- [GameObject](gameobject.md) — what the API looks like: transforms, tags, hierarchy, cloning.
- [Components](components/index.md) — how components work, how their lifecycle runs.
- [Component Lifecycle](components/component-lifecycle.md) — `OnAwake`, `OnStart`, `OnUpdate`, `OnFixedUpdate`, the rest of the hooks.
- [Component Reference](components/reference/index.md) — the catalog of every built-in component.

After that:

- [Prefabs](prefabs/index.md) — saving GameObject hierarchies as reusable templates.
- [Scenes](scenes/index.md) — what a scene file actually is, scene metadata, runtime tracing.
- [Maps](maps/index.md) — when a "level" is a separately-loaded asset rather than your gameplay scene.
- [GameObjectSystem](gameobjectsystem.md) — for engine-style global systems that need to run between specific frame phases.
- [Scene Mapping](scene-mapping/index.md) — building level geometry directly inside the scene editor.

## Things that will trip you up

**Disabled parents disable their children.** Toggling a parent off stops everything underneath from ticking and rendering. The child's *local* `Enabled` flag is preserved, so re-enabling the parent doesn't accidentally turn on children you'd previously turned off.

**Physics owns the transform on simulated Rigidbodies.** If a `Rigidbody` is dynamic/simulated and you set `WorldPosition` directly, you're racing the physics step. The result is visual jitter at best, broken physics at worst. Apply forces or velocities instead, or set `Rigidbody.Velocity` directly.

**`Components.GetInDescendants<T>` and `Scene.GetAllComponents<T>` are slow.** They walk the tree every call. Cache the result in `OnStart` if you need it in `OnUpdate`. The faster patterns: `[RequireComponent]` on a sibling property, or `Components.Get<T>()` (this object only) cached once.

**Renderer with no Camera = nothing.** A `ModelRenderer` without a `CameraComponent` somewhere in the scene renders to nothing visible. The default scene template gives you a camera; if you build a scene from scratch in code, add one.

## Related Pages

- [Architectural Overview](../architecture.md) — where the Scene System sits in the engine
- [Editor](../editor/index.md) — the tools you use to author scenes
- [Sweeper Sample](../build-games/samples-templates/sweeper.md) — a small, complete scene to read
