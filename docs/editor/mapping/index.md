---
title: "Mapping (Legacy Hammer)"
icon: "🔨"
created: 2026-02-16
updated: 2026-04-27
sources:
  - engine/Sandbox.Engine/Scene/Components/Map/HammerMesh.cs
---

# Mapping (Legacy Hammer)

> **For new projects, don't use Hammer.** The recommended path is [Scene Mapping](../../scene/scene-mapping/index.md) — `.scene`-as-map with [`MeshComponent`](../../scene/components/reference/meshcomponent.md) + [`PolygonMesh`](../../scene/components/reference/polygonmesh.md), driven by the in-scene `MeshTool` (Block Tool, `Shift+B`). This page covers the **legacy Hammer mapping** flow that ships compiled `.vmap` files into the engine.

## What "Hammer mapping" means

Hammer is the external level editor inherited from Source. It produces `.vmap` files: brush- and mesh-based geometry authored outside the s&box scene editor and compiled into the engine at build time.

In the s&box runtime, Hammer-authored geometry doesn't live as scene meshes you can edit — it's loaded as compiled models attached to GameObjects, with each piece of geometry that was *tied to an entity* in Hammer surfacing as a GameObject carrying a [`HammerMesh`](../../scene/components/reference/hammer-mesh.md) component. That component is `[Hide]`d in the Inspector by design and **populated automatically at compile time**; you don't add it manually from the Add Component menu.

## End-to-end flow

1. **Author in Hammer.** Open the `.vmap` in the Hammer editor and build brushes / meshes there. Apply materials, set up entities, tie geometry to entities where you need a runtime GameObject.
2. **Compile.** The engine compiles the `.vmap` into a map asset. Each tied piece becomes a GameObject; each gets a `HammerMesh` component with its `Model` populated and (by default) a hidden `ModelRenderer` and `ModelCollider` set up for it.
3. **Use it from C#.** At runtime you can find these GameObjects, read or change the `HammerMesh.Tint`, `SurfaceVelocity`, `IsTrigger`, etc. — see the [`HammerMesh` component reference](../../scene/components/reference/hammer-mesh.md) for what's exposed and what each property means.

The most common reason to touch `HammerMesh` from code is to wire trigger volumes (`IsTrigger` + `OnTriggerEnter`/`OnTriggerExit`) or surface effects (`SurfaceVelocity` for conveyor belts, `Friction`/`Surface` for gameplay tweaks).

## Why this is being phased out

Hammer's compile step keeps map geometry isolated from the rest of the scene — you can't edit a brush from the Inspector, prefab a wall, or hot-reload a layout change without recompiling. Scene Mapping removes that wall: maps are `.scene` files, geometry is normal scene data, and the same tooling that edits everything else also edits levels.

For new work, start with [Scene Mapping](../../scene/scene-mapping/index.md). Use this page only when you're maintaining a project that already ships a Hammer-authored `.vmap`.

## Related Pages

- [Scene Mapping](../../scene/scene-mapping/index.md) — recommended modern path
- [`HammerMesh` component](../../scene/components/reference/hammer-mesh.md) — runtime API for compiled Hammer geometry
- [`MeshComponent`](../../scene/components/reference/meshcomponent.md) / [`PolygonMesh`](../../scene/components/reference/polygonmesh.md) — building blocks of in-scene mapping
- [Colliders](../../systems/physics/colliders.md)
