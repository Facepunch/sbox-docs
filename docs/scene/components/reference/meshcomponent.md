---
title: "Mesh Component"
icon: "🥽"
created: 2026-04-25
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Scene/Components/Mesh/MeshComponent.cs
  - engine/Sandbox.Engine/Scene/Components/Mesh/PolygonMesh.cs
---

# Mesh Component

> **New to mapping?** This is the reference for the runtime mesh component. The authoring tool that creates these is documented at **[Scene Mapping](../../scene-mapping/index.md)**.

> **Which should I use — `MeshComponent` or [`HammerMesh`](hammer-mesh.md)?**
>
> | Use | Component | Authored by |
> |---|---|---|
> | **New project, scene-first mapping** | `MeshComponent` (this page) | The in-scene [Mapping](../../scene-mapping/index.md) tool, with [`PolygonMesh`](polygonmesh.md) data |
> | **Legacy `.vmap` map, brushes tied to entities** | [`HammerMesh`](hammer-mesh.md) | Hammer's "tie to entity" workflow at `.vmap` compile time |
>
> If you're starting fresh, you want `MeshComponent`. If you're working with an existing Hammer map, you'll see `HammerMesh` on tied-mesh entities.

`MeshComponent` is the runtime component that owns an editable [`PolygonMesh`](polygonmesh.md) and produces a renderable model + a collider. It's the runtime representation of geometry built with the in-scene **Mapping** tool — see [Scene Mapping](../../scene-mapping/index.md) for the authoring workflow.

> `MeshComponent` is `[Hide]`-marked and added automatically by the Mapping tool's primitive editors (Block, Cylinder, etc.). You don't drop one onto a GameObject manually. Use the Mapping tool from the editor toolbar to create geometry; the component appears as part of that flow.

## Properties

| Property | Type | Default | Description |
|---|---|---|---|
| `Mesh` | `PolygonMesh` | — | The owned editable mesh. `[Hide]` in the inspector — managed by the Mapping tool. |
| `Collision` | `CollisionType` | `Mesh` | `None`, `Mesh` (full per-triangle collision), or `Hull` (convex hull). Setting this rebuilds the collider. |
| `Color` | `Color` | `White` | Tint applied to the rendered geometry. Stored as `Color` (with `[Title("Tint")]`). |
| `SmoothingAngle` | `float` | `0` | Auto-smooth threshold for vertex normals. Applied via `Mesh.SetSmoothingAngle`. |
| `HideInGame` | `bool` | `false` | When true, the component renders in the editor but not at runtime — useful for designer-only volumes (think no-clip walls, viz markers). |
| `RenderType` | `ShadowRenderType` | `On` | `[Title("Cast Shadows")]`, `[Category("Lighting")]`. Standard shadow control: `On`, `Off`, `ShadowsOnly`. |
| `Model` *(read-only)* | `Model` | — | The runtime `Model` produced from `Mesh`. `[JsonIgnore, Hide]`. |
| `IsConcave` *(override)* | `bool` | — | True when `Collision == CollisionType.Mesh`. Lets the physics system decompose the mesh accordingly. |

## Common Methods

```csharp
// Per-face material assignment
mesh.SetMaterial( material, triangleIndex );
var mat = mesh.GetMaterial( triangleIndex );
```

`SetMaterial`/`GetMaterial` operate on triangle indices; internally they resolve to the underlying `PolygonMesh` face.

## Lifecycle

- **`OnEnabledInternal`** — adds `"world"` to `GameObject.Tags`, builds the render mesh, then defers to the base `Collider.OnEnabledInternal` so collision sets up after geometry exists.
- **`OnDisabled`** — destroys the active `SceneObject`.
- **`HideInGame` toggle** — at runtime, flips between rendering and not. Has no effect in the editor.

## Common Patterns

### Trigger volumes from MeshComponent

Build a volume in the Mapping tool, then on the resulting `MeshComponent`:

1. Set `Collision = CollisionType.Mesh` (or `Hull` if a convex shape is acceptable — much faster).
2. On the inherited `Collider`, set `IsTrigger = true`.
3. Add `Tags` for filtering.
4. Subscribe `OnTriggerEnter` / `OnTriggerExit` from a sibling component.

### Editor-only visualization

Set `HideInGame = true` to use the geometry as designer guides (volumes for AI, no-go zones, etc.) without rendering it to players. The component still simulates collision unless you also set `Collision = None`.

### Per-face material swaps at runtime

```csharp
// React to a hit and re-paint the face that was hit
public void OnHit( SceneTraceResult tr )
{
    if ( tr.Component is MeshComponent mc && tr.Triangle >= 0 )
    {
        mc.SetMaterial( BurnedMaterial, tr.Triangle );
    }
}
```

- **`MeshComponent` is `[Hide]`.** It won't appear in the standard "Add Component" menu. The Mapping tool adds it for you, or you reveal it via the Inspector's "show hidden" toggle.
- **Concave collision is expensive.** `CollisionType.Mesh` triggers per-triangle collision detection. For static-but-shaped geometry that doesn't need exact collision, prefer `Hull`. For pure visual geometry, set `Collision = None`.
- **`Mesh` reassignment rebuilds.** Setting the `Mesh` property to a different `PolygonMesh` triggers a full `RebuildMesh(true)`. Mutating the existing `PolygonMesh` in-place doesn't auto-rebuild — the editor handles this internally; in code, you typically don't mutate the mesh outside the editor.
- **JSON serialization.** `MeshComponent` carries `Mesh` as `[Property, Hide]`, so the `PolygonMesh` is part of the saved scene/prefab. Large meshes inflate `.scene` size; the `PolygonMesh` JSON serializer compresses the topology with gzip+base64.

## Related Pages

- [Scene Mapping](../../scene-mapping/index.md) — the authoring workflow that produces `MeshComponent`s
- [Polygon Mesh](polygonmesh.md) — the editable mesh data type owned by this component
- [Hammer Mesh](hammer-mesh.md) — the older Hammer-driven equivalent (different component)
- [Colliders](../../../systems/physics/colliders.md) — the base type that `MeshComponent` extends
