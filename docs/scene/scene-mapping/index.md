---
title: "Scene Mapping"
icon: "🔧"
created: 2026-04-25
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Scene/Components/Mesh/PolygonMesh.cs
  - engine/Sandbox.Engine/Scene/Components/Mesh/MeshComponent.cs
  - game/addons/tools/Code/Scene/Mesh/Tools/MeshTool.cs
---

# Scene Mapping

**Scene Mapping** (officially the **Mapping** tool inside the scene editor — `[Title("Mapping")]` on `MeshTool` in `game/addons/tools/Code/Scene/Mesh/Tools/MeshTool.cs`) is s&box's in-scene level-authoring system. It replaces the Hammer-driven `.vmap` workflow: you build geometry, lighting, gameplay components, and prefabs **directly in the scene editor**, save the result as a `.scene` file, and load it at runtime via [`MapInstance`](../components/reference/mapinstance.md).

It's the modern path. New projects should start here.

## Why Scene Mapping over Hammer

| Concern | Hammer (`.vmap`) | Scene Mapping (`.scene`) |
|---|---|---|
| Authoring environment | Separate Hammer window | The same scene editor you use for gameplay |
| Saved as | `.vmap` → compiled `.vpk` | `.scene` (text JSON, no compile step) |
| Brushwork | Solid brushes | [`MeshComponent`](../components/reference/meshcomponent.md) + [`PolygonMesh`](../components/reference/polygonmesh.md) (half-edge, fully editable at any time) |
| Entities | FGD `[HammerEntity]` definitions | Plain `Component`s on plain `GameObject`s |
| Iteration | Compile, reload | Save scene, reload (or hotload) |
| Prefabs | Limited | First-class — drop any prefab into the level |
| Mixing with gameplay | Always two worlds | One scene tree, one editor |

Hammer still ships and is supported for legacy `.vmap` projects, but the engine's level authoring is converging on the Mapping toolset. See [Hammer (Mapping)](../../editor/hammer.md) for the deprecation note.

## How It Works

The Mapping tool operates on `MeshComponent`s — components that hold a `PolygonMesh` (an editable half-edge mesh) plus collision settings. You select the Mapping tool from the editor toolbar, drop into one of its subtools, and edit geometry directly on the GameObject the mesh lives on.

The runtime types involved:

- **`PolygonMesh`** (`engine/Sandbox.Engine/Scene/Components/Mesh/PolygonMesh.cs`) — the editable polygon mesh. Half-edge topology, faces, edges, vertices, materials, UVs. Serialized as JSON inside the `.scene` file.
- **`MeshComponent`** (`engine/Sandbox.Engine/Scene/Components/Mesh/MeshComponent.cs`) — the runtime component that owns a `PolygonMesh` and produces a renderable model + a collider. `[Hide]`-marked because the Mapping tool adds it for you; you don't drop it manually.

When the scene is loaded by `MapInstance` (with a `.scene` `MapName`), each mesh is rebuilt from its `PolygonMesh` data the same way it was authored.

## Subtools

`MeshTool.GetSubtools()` returns the active subtool list. Each subtool changes what mouse interactions do.

| Subtool | Source | Purpose |
|---|---|---|
| **Primitive** | `Tools/PrimitiveTool.cs` + `BlockEditor`, `PolygonEditor`, `CableEditor` | Block out new geometry — drag-create boxes, cylinders, polygons, cables. The `BlockEditor` is the most common entry point. |
| **Object Selection** | `Tools/ObjectSelection.cs` | Pick an entire `MeshComponent` to operate on at the GameObject level. |
| **Vertex** | `Tools/VertexTool.cs` | Move/snap/weld individual vertices. |
| **Edge** | `Tools/EdgeTool.cs` | Move/extrude/cut edges; access EdgeCut, EdgeArch, Bridge, Bevel modifiers. |
| **Face** | `Tools/FaceTool.cs` | Select faces (with optional Select-By-Material / Select-By-Normal filters), extrude, inset, cut, apply materials. |
| **Texture** | `Tools/TextureTool.cs` | Edit per-face material assignments and UV layout (planar projection, alignment). |
| **Vertex Paint** | `Tools/VertexPaintTool.cs` | Paint vertex colors / blend weights. |
| **Displacement** | `Tools/DisplacementTool.cs` | Convert flat surfaces into displaced terrain-like meshes. |

Modifiers invoked from within the subtools include:

- `BevelTool` — round edges
- `BridgeTool` — connect two open edge loops with new geometry
- `ClipTool` — slice a mesh by a plane
- `EdgeArchTool` — convert a straight edge into an arch
- `EdgeCutTool` — insert new edges (loop cuts)
- `MirrorTool` — symmetric editing

## Building a Level: Workflow

1. Open the scene editor.
2. Select the **Mapping** tool from the top toolbar (`[Title("Mapping")]`, hammer-and-wrench icon).
3. Drop into the **Primitive** subtool, pick **Block**, drag a box. The editor creates a `GameObject` with a `MeshComponent` holding the new `PolygonMesh`.
4. Use **Face** / **Edge** / **Vertex** subtools to refine the geometry.
5. Add a material via the **Texture** subtool.
6. Add lights, props (regular `ModelRenderer` GameObjects), trigger volumes (`MeshComponent` with `Collision = Mesh` + `IsTrigger = true`), gameplay components, prefabs.
7. Save the scene as `levels/myarena.scene`.
8. Load it via `MapInstance.MapName = "levels/myarena.scene"`.

## Loading a Scene Map at Runtime

```csharp
using Sandbox;

public sealed class LevelLoader : Component
{
    [Property] public string LevelPath { get; set; } = "levels/myarena.scene";

    protected override void OnStart()
    {
        var go = new GameObject( true, "Map" );
        go.SetParent( GameObject );

        var map = go.Components.Create<MapInstance>();
        map.OnMapLoaded += () => Log.Info( "Scene map loaded" );
        map.MapName = LevelPath;
    }
}
```

`MapInstance` recognizes the `.scene` extension and routes through its scene-map loader. See [Loading Maps](../maps/loading-maps.md) for the full list of accepted `MapName` shapes.

## Common Patterns

1. **Trigger volumes via `MeshComponent`.** Build the volume geometry like any block, then on the `MeshComponent` set `Collision = Mesh` and toggle `IsTrigger`. Use `OnTriggerEnter` / `OnTriggerExit` for events.
2. **Static cover with prefabs over brushwork.** Block out gameplay-relevant volumes (cover, ramps, doorways) with the Mapping tool. Drop pre-made prop prefabs on top for visual detail.
3. **Light bake from the scene.** Place lights as regular GameObjects with `Light` components. Bakes can run from the scene; you don't need a Hammer compile pipeline.
4. **Mixing with Hammer.** A `.scene` map can still reference a Hammer-authored `.vpk` if you want to import existing geometry — though for new work, building it natively in Scene Mapping is cleaner.

- **`MeshComponent` is `[Hide]`.** You don't drop it manually — the Mapping tool's primitive editors create it for you. Inspector reveals it via the standard "show hidden" toggle if you need to inspect properties.
- **Mesh serialization is JSON.** A complex map's `.scene` file can grow large. The mesh data is base64+gzip-compressed inside the JSON for efficiency; see `PolygonMesh.Serialize.cs` for the legacy/current formats.
- **Half-edge topology.** Internally `PolygonMesh` is a half-edge mesh, so manifold operations (bevel, bridge, clip) are well-defined; non-manifold operations may fail silently or produce degenerate geometry. Stick to clean closed surfaces.
- **No baked navmesh from raw Mapping output.** `MeshComponent` collision feeds the runtime navmesh generator; if you need pre-baked navmesh, use the navmesh tooling on the assembled scene rather than expecting Hammer's compile-time bake.

## Related Pages

- [Maps overview](../maps/index.md) — runtime map loading and entities
- [Loading Maps](../maps/loading-maps.md) — the `.scene` `MapName` shape and load flow
- [Hammer (Mapping)](../../editor/hammer.md) — legacy alternative
- [Hammer Mesh component](../components/reference/hammer-mesh.md) — bridge component for Hammer-authored brush geometry
- [Scene System](../index.md)
