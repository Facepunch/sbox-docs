---
title: "Polygon Mesh"
icon: "📐"
created: 2026-04-25
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Scene/Components/Mesh/PolygonMesh.cs
  - engine/Sandbox.Engine/Scene/Components/Mesh/PolygonMesh.Bevel.cs
  - engine/Sandbox.Engine/Scene/Components/Mesh/PolygonMesh.Clip.cs
  - engine/Sandbox.Engine/Scene/Components/Mesh/PolygonMesh.EdgeSpan.cs
  - engine/Sandbox.Engine/Scene/Components/Mesh/PolygonMesh.Serialize.cs
  - engine/Sandbox.Engine/Scene/Components/Mesh/PolygonMesh.Subdivision.cs
  - engine/Sandbox.Engine/Scene/Components/Mesh/PolygonMesh.UV.cs
---

# Polygon Mesh

> **New to mapping?** This is the reference for the underlying mesh data type. The authoring tool that edits these is documented at **[Scene Mapping](../../scene-mapping/index.md)**.

`PolygonMesh` is the editable half-edge mesh data type owned by a [`MeshComponent`](meshcomponent.md). It's the underlying representation that the in-scene **Mapping** tool edits — see [Scene Mapping](../../scene-mapping/index.md) for the authoring workflow.

`PolygonMesh` is **not a Component** itself — it's a serializable data class (`IJsonConvert`-implementing). You don't drop it onto a GameObject. It lives inside `MeshComponent.Mesh`, gets edited through the Mapping subtools, and is serialized into your `.scene` file.

## Public Surface

`PolygonMesh` is `[Expose]`-marked, so reflection-based UIs can interact with it, but most game code doesn't touch it directly — you author through the Mapping tool, save the `.scene`, and let the engine roundtrip it.

The most commonly-used members:

| Member | Purpose |
|---|---|
| `Transform` | World-space transform of the mesh, separate from the GameObject's transform. |
| `IsDirty` | Set when topology or materials have changed; the Mapping tool rebuilds when this flips. |
| `VertexHandleFromIndex(int)` / `HalfEdgeHandleFromIndex(int)` / `FaceHandleFromIndex(int)` | Resolve a topology handle from a stable index. Useful for serialization and tools. |
| `MergeMesh(source, transform, ...)` | Merge another `PolygonMesh` into this one. |
| `BevelEdges(edges, mode, segments, distance, shape, ...)` | Bevel a set of edges. `BevelEdgesMode` controls how original edges are kept or removed. |
| `SetSmoothingAngle(angle)` | Auto-compute smoothing groups for shared vertices below the angle threshold. |
| `SetFaceMaterial(face, material)` / `GetFaceMaterial(face)` | Per-face material assignment. |
| `TriangleToFace(triangleIndex)` | Resolve a triangulated render-mesh triangle back to its source face. |

## Operations (one partial file each)

| File | What it implements |
|---|---|
| `PolygonMesh.cs` | Core topology, materials, UV parameters, render-mesh build |
| `PolygonMesh.Bevel.cs` | Bevel — `BevelEdges` with multi-segment / shape control |
| `PolygonMesh.Clip.cs` | Clip — slice a mesh by a plane (used by the Clip Tool) |
| `PolygonMesh.Subdivision.cs` | Subdivide — Catmull-Clark-style face subdivision |
| `PolygonMesh.UV.cs` | UV generation / face-vertex UV updates / projection modes |
| `PolygonMesh.EdgeSpan.cs` | Edge-span resolution helpers used by edge-loop tools |
| `PolygonMesh.Serialize.cs` | JSON read/write — supports the legacy and current scene formats |

## Serialization

`PolygonMesh` implements `IJsonConvert`. The serializer:

- Stores the half-edge topology as **gzip-compressed, base64-encoded** data inside the JSON (the `"Topology"` key).
- Carries per-face texture parameters, materials by name, and the mesh `Transform`.
- Has a back-compat path for the older format that used per-vertex `Position`/`Rotation` + a face-vertex list (`PolygonMesh.Serialize.cs` references the legacy form via `Topology` + `Position` + `Rotation` + `Positions`).

A `MeshComponent` with a non-trivial `PolygonMesh` adds tens of KB to the enclosing `.scene` file. Very large maps may reach hundreds of KB per mesh; this is normal.

- **Manifold-only operations.** Bevel, bridge, and loop-cut assume manifold topology. Non-manifold meshes (T-junctions, duplicated geometry, three-faces-on-one-edge) may corrupt or no-op.
- **Triangulation is built lazily.** The list of `_triangleFaces` and `_meshIndices` is only valid after a build pass. If you read these directly outside the editor, expect them to be empty or stale.
- **Smoothing angle is per-mesh, not per-face.** A single threshold across the whole mesh; if you need different smoothing per region, split into multiple meshes.
- **Material id reuse.** Materials are interned by name in `_materialIdsByName` — assigning the same material reference repeatedly does not bloat the material table.

## Related Pages

- [Mesh Component](meshcomponent.md) — the runtime component that owns a `PolygonMesh`
- [Scene Mapping](../../scene-mapping/index.md) — the authoring workflow that edits `PolygonMesh`
- [Hammer Mesh](hammer-mesh.md) — the older Hammer-driven mesh component (separate type)
