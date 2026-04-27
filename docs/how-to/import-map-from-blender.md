---
title: "Import a Map from Blender"
icon: "📦"
created: 2026-04-27
updated: 2026-04-27
sources:
  - engine/Sandbox.Tools/ModelEditor/ModelDoc.cs
  - engine/Sandbox.Engine/Scene/Components/Render/ModelRenderer.cs
  - engine/Sandbox.Engine/Scene/Components/Collider/ModelCollider.cs
  - engine/Sandbox.Engine/Scene/Components/Game/Prop.cs
---

# Import a Map from Blender

You authored a level (or a building, or a chunk of geometry) in Blender and want it inside an s&box game. This page covers the end-to-end import path: Blender → FBX → ModelDoc → scene.

## Pick the right path first

s&box has three ways geometry ends up in a scene. For pre-built Blender geometry, **only one is right**:

| Path | What it is | Use this for Blender import? |
|---|---|---|
| **Imported model** (`.vmdl` via [ModelDoc](../editor/model-editor.md)) | Compile your FBX into a model asset, place it in the scene with `ModelRenderer` + `ModelCollider`. | ✅ **Yes — this page** |
| **Scene Mapping** ([`MeshComponent`](../scene/components/reference/meshcomponent.md) + [`PolygonMesh`](../scene/components/reference/polygonmesh.md)) | Author geometry directly inside s&box using the in-scene `MeshTool`. | ❌ This is an authoring tool, not an importer. |
| **Legacy Hammer** ([`.vmap`](../editor/mapping/index.md)) | Build geometry in the external Hammer editor and compile. | ❌ Not for FBX import. Hammer authors its own brushes/meshes. |

A "map" from Blender is conceptually a model — possibly a very large one. You'll import it as a `.vmdl` and either spawn it as a static prop or place its `ModelRenderer` directly under your scene root.

## Step 1 — Export from Blender

In Blender's `File → Export → FBX (.fbx)` dialog, set:

| Setting | Value | Why |
|---|---|---|
| **Path Mode** | Copy (with embed textures if you want them inline) | Keeps texture references resolvable. |
| **Apply Scalings** | `FBX Units Scale` or `All Local` | s&box works in **inches** (≈ 39.37 per metre). If you author in metres in Blender, applying scaling at export is the simplest path; otherwise you'll scale in ModelDoc. |
| **Forward** / **Up** | `-Y Forward`, `Z Up` (Blender's default suits the FBX importer) | s&box is Z-up too; the default mapping preserves orientation. |
| **Apply Transform** | ✅ enabled | Bakes object transforms into the mesh — avoids "everything is at the origin" or "everything is rotated 90°" surprises. |
| **Selected Objects** | ✅ if you only want what's selected | Keeps the export tight. |
| **Object Types** | Mesh (Empty / Armature only if you actually need them) | Don't export cameras or lights. |
| **Mesh → Smoothing** | `Face` or `Edge` (not `Off`) | Without this, normals come in unsmoothed and shading looks faceted. |
| **FBX format** | Binary (the default) | s&box doesn't accept ASCII FBX. |

Material naming gotcha: Blender suffixes duplicate names with `.001`, `.002`, etc. The FBX importer treats anything after a period as a discard, which collapses `Brick.001` and `Brick.002` into the same `Brick` slot. Rename them to unique strings before exporting if you need them separate.

## Step 2 — Import into s&box

1. Drop the `.fbx` into the **Asset Browser** at the location where you want the model to live (e.g. `models/maps/your_map/`).
2. The editor opens **ModelDoc**, with a `MeshList` node already pointing at your FBX.
3. Save. ModelDoc compiles a `.vmdl` next to the source FBX.

If the geometry comes in tiny or huge, add a **`ScaleAndMirror`** node in ModelDoc rather than fighting Blender's export — Facepunch's own citizen pipeline does this (sources are in cm, the model is rescaled in ModelDoc, see [Citizen Characters](../assets/ready-to-use-assets/citizen-characters.md)).

## Step 3 — Materials

Each FBX material slot becomes a slot on the compiled `.vmdl`. ModelDoc shows them in the model's material list. To assign s&box materials:

- If the FBX referenced texture files that the importer found, you'll get auto-generated `.vmat` stubs.
- Otherwise, create a `.vmat` per slot (`Asset Browser → Right-click → New → Material`) and assign it in ModelDoc's material list.
- If you swap materials per-instance later, that's done at runtime via [`ModelRenderer.MaterialOverride`](../scene/components/reference/index.md) — but for a map, baking the materials into the `.vmdl` is what you want.

## Step 4 — Author collision

A rendered mesh has *no* physics by default. For a map, you usually need collision so players don't fall through it.

In ModelDoc, add a **physics node** that points at your mesh. The two main choices:

- **Per-mesh / per-face physics shapes** — accurate; suitable when the mesh is already low-poly enough to use directly. Heaviest at runtime.
- **Concave hull / convex decomposition** — generates simplified collision automatically. Best for complex map geometry; tune the decomposition parameters until the result is faithful enough.

For a *map* (concave geometry players move around inside), per-face physics on the visual mesh is usually fine if poly count is reasonable. If the visible geometry has a million tris, author a separate, simplified collision mesh in Blender and point the physics node at *that*.

If you skip this step, the model loads but has no collider — the `ModelCollider` you add at Step 5 will silently do nothing because there's no physics data in the `.vmdl` for it to read.

## Step 5 — Place it in the scene

1. Create a `GameObject` in your scene (or use an existing one) for the map.
2. Add a **`ModelRenderer`** and set its `Model` to the new `.vmdl`.
3. Add a **`ModelCollider`** on the same GameObject. It will pick up the model from the `ModelRenderer` automatically — see [`ModelCollider.cs:32-35`](https://github.com/Facepunch/sbox-public).
4. If the map is static (not moving, not animated), tick **`Static`** on the `ModelCollider` for cheaper physics.

Alternatively, drop a **[`Prop`](../scene/components/reference/prop.md)** component on the GameObject — that bundles `ModelRenderer` + `ModelCollider` + a few sensible defaults in one place, useful for level pieces you spawn dynamically.

## Common pitfalls

- **"It's invisible / black."** Material slots aren't bound. Open the `.vmdl` in ModelDoc and check that each material shows a real `.vmat`, not the magenta error material.
- **"It's tiny / massive."** Scale mismatch between Blender and s&box. Easiest fix: add a `ScaleAndMirror` in ModelDoc. (s&box uses inches; 1 m ≈ 39.37 inches.)
- **"Players fall through."** Either no physics node was added in ModelDoc, or `ModelCollider` is missing on the scene GameObject. Both are needed.
- **"Lighting / shading looks faceted."** Smoothing wasn't exported — re-export the FBX with `Mesh → Smoothing = Face` or `Edge`.
- **"Materials all collapsed into one."** Blender's `.001` / `.002` suffixes — rename them to unique strings and re-export.
- **"Wrong orientation / rotated 90°."** Re-export with `Apply Transform` ticked and confirm Blender's Forward = `-Y`, Up = `Z`.

## Related pages

- [Model Editor (ModelDoc)](../editor/model-editor.md) — the editor you'll spend most of the import time inside.
- [Custom Assets](../assets/custom-assets.md) — how `.vmdl` fits into the broader asset pipeline.
- [Hammer Mesh](../scene/components/reference/hammer-mesh.md) — the legacy alternative if you're maintaining a `.vmap`-based map.
- [Scene Mapping](../scene/scene-mapping/index.md) — building geometry inside s&box instead of importing it.
