---
title: "Import a Map from Blender"
icon: "📦"
created: 2026-04-27
updated: 2026-04-27
sources:
  - engine/Sandbox.Tools/ModelEditor/ModelDoc.cs
  - engine/Definitions/modeldoc/ModelDocApp.def
  - engine/Sandbox.Engine/Scene/Components/Render/ModelRenderer.cs
  - engine/Sandbox.Engine/Scene/Components/Collider/ModelCollider.cs
  - engine/Sandbox.Engine/Scene/Components/Game/Prop.cs
---

# Import a Map from Blender

You authored a level in Blender and want it inside an s&box game. The **import** step is almost trivial — drop the FBX in. The interesting work is what you wire up inside **ModelDoc** afterwards, and especially how you choose collision.

## Pick the right path first

Three ways geometry can end up in a scene; for pre-built Blender geometry, only one applies:

| Path | What it is | Use this for Blender import? |
|---|---|---|
| **Imported model** (`.vmdl` via [ModelDoc](../editor/model-editor.md)) | Compile the FBX into a `.vmdl`, place with `ModelRenderer` + `ModelCollider`. | ✅ **Yes — this page** |
| **Scene Mapping** ([`MeshComponent`](../scene/components/reference/meshcomponent.md) + [`PolygonMesh`](../scene/components/reference/polygonmesh.md)) | Author geometry directly inside s&box using the in-scene `MeshTool`. | ❌ Authoring tool, not an importer. |
| **Legacy Hammer** ([`.vmap`](../editor/mapping/index.md)) | Author in the external Hammer editor and compile. | ❌ Not for FBX import. |

A "map" from Blender is conceptually a (possibly very large) model. You'll compile it to a `.vmdl` and place it in the scene like any other prop.

## Step 1 — Export from Blender

For the vast majority of static map geometry, **Blender's default FBX export settings are fine**. Drop the file in and move on. The only export details that ever matter:

- **FBX format**: Binary (Blender's default). s&box won't read ASCII FBX.
- **Mesh smoothing**: set to `Face` or `Edge` (not `Off`) if your shading looks faceted.
- **Material naming**: Blender appends `.001`, `.002` to duplicated names. The FBX importer truncates anything after the period, collapsing them into one slot. Rename if you actually need them separate.

You only need to think harder than that when:

- **The mesh is skinned / animated** — bone rotations, rest pose, animation channels matter.
- **You're porting from another engine** — axis convention and unit scale will differ.
- **Scale is wrong on import** — see Step 2; usually easier to fix in ModelDoc than to fight Blender.

For scale: s&box uses **inches** (39.37 ≈ 1 metre). Don't try to bake the conversion into Blender; do it in ModelDoc instead (next section).

## Step 2 — Inside ModelDoc

Drop the `.fbx` into the **Asset Browser** at the destination path (e.g. `models/maps/your_map/`). The editor opens **ModelDoc** with a graph already containing a `RenderMeshFile` node pointing at the FBX. Save once and a `.vmdl` is compiled next to the source.

What you'll typically wire up after the initial import:

| Node / concern | Purpose |
|---|---|
| **`RenderMeshFile`** | Loads the FBX (already added). One per mesh file you want to include. |
| **`MaterialGroup`** | Override per-slot materials, or define alternate "skins" on the same model. |
| **Scale & Mirror** | Fix unit mismatch (cm → inches) or hand-edness. Preferred over fighting Blender export — Facepunch's Citizen pipeline does exactly this (see [Citizen Characters](../assets/ready-to-use-assets/citizen-characters.md)). |
| **LOD nodes** | Author lower-detail variants for distance-based switching. Critical for large maps that fill the screen. |
| **Attachments** | Named transforms an outside system can query (e.g. spawn points, light hookups). |
| **Physics nodes** | Authoring collision — see Step 3. |

## Step 3 — Authoring physics

This is where Blender map imports get nuanced. ModelDoc gives you several ways to derive physics from your render mesh, with different cost/accuracy trade-offs:

### Physics Mesh from Render (`PhysicsMeshFromRender`)

Copies the visible triangle mesh into a triangle physics mesh.

- **Accuracy**: ideal — collision matches what you see exactly.
- **Cost**: high. Triangle meshes are the most expensive collision shape; raycasts and overlap queries pay per-triangle.
- **Restrictions**: triangle meshes are **static**. You can't make a `prop_physics` that moves and bounces with this kind of collision.

Right for: static map geometry where you can afford the cost and you don't want to think about it. Most maps that are "just an environment" land here.

### Physics Hull from Render (`PhysicsHullFromRender`)

Generates one or more **convex hulls** from the render mesh. Has a `HullMode` (e.g. *hull per element*) controlling how the source mesh is partitioned into hulls.

- **Accuracy**: lossy. Hulls are convex by definition; concave details (alcoves, indentations, openings) get filled in.
- **Cost**: low. Hulls are the cheapest moving-collision shape.
- **Restrictions**: works for both static and dynamic objects.

Right for: anything that needs to move, anything spawned in large numbers, or any map where the per-tri cost of a triangle mesh would hurt. **Preferred default for performance**, but only useful if the result is faithful enough — which depends on how the mesh is split into hulls.

### Manual / authored hulls

For maximum control, author the collision shapes yourself:

- In Blender, model **separate convex meshes** for collision and place them in the same FBX. Common convention: prefix the collision objects (`_phys`, `_collision`) and reference them from a dedicated physics node in ModelDoc.
- Or feed multiple `RenderPrimitiveMesh` nodes through `PhysicsHullFromRender` with the hull mode that matches your authoring (one hull per element vs. merged).

This is the **preferred but advanced** path: the right answer for a "real" map is rarely a single convex hull or a million-tri triangle mesh, but a hand-decomposed set of convex pieces. Don't reach for it until the simpler nodes have actually disappointed you.

### Per-mesh physics hint

`RenderPrimitiveMesh` has a `PhysicsHint` (`NONE` / `MESH` / `HULL` / `ALL`) you can set per individual mesh element. Use it to keep a render-only mesh in the model (decorative trim) while still getting collision from the structural meshes.

### Heuristic: which to pick

- Static, non-critical, "this is a wall and a floor" → **Physics Mesh from Render**. Stop thinking.
- Lots of instances, or anything that should ever move → **Physics Hull from Render**.
- A serious shipping map that you'll profile → **manual hulls** for the load-bearing geometry, hulls-from-render for the rest, render-only for decoration.

If you skip authoring any physics node at all, the model loads but has no collider data — `ModelCollider` in the scene will silently do nothing.

## Step 4 — Place it in the scene

1. Create a `GameObject` for the map.
2. Add a **`ModelRenderer`** and set its `Model` to the new `.vmdl`.
3. Add a **`ModelCollider`** on the same GameObject. It picks up the model from the `ModelRenderer` automatically (`ModelCollider.cs:32-35`).
4. If the map doesn't move (almost always true), tick **`Static`** on the `ModelCollider` for cheaper physics.

For dynamic level pieces or pickups you spawn at runtime, drop a [`Prop`](../scene/components/reference/prop.md) instead — that bundles `ModelRenderer` + `ModelCollider` with sensible defaults.

## Common pitfalls

- **"It's invisible / black."** Material slots aren't bound. Open the `.vmdl` in ModelDoc and confirm each slot has a real `.vmat`, not the magenta error material.
- **"It's tiny / massive."** Scale mismatch. Add a Scale & Mirror node in ModelDoc (39.37 inches = 1 metre).
- **"Players fall through."** No physics node was authored, or `ModelCollider` is missing on the scene GameObject. Both are needed.
- **"Performance tanks when I add the map."** You're using `PhysicsMeshFromRender` on a high-poly mesh, or running raycasts/overlaps frequently against it. Switch to `PhysicsHullFromRender` for anything that doesn't strictly need triangle accuracy.
- **"My collision is convex / fills in openings."** That's `PhysicsHullFromRender` doing its job. Either switch to `PhysicsMeshFromRender`, or split the mesh into convex pieces and use `hull-per-element` mode.
- **"Lighting / shading looks faceted."** Re-export the FBX with `Mesh → Smoothing = Face` or `Edge`.
- **"Materials all collapsed into one."** Blender's `.001` / `.002` suffix collision. Rename and re-export.

## Related pages

- [Model Editor (ModelDoc)](../editor/model-editor.md)
- [Custom Assets](../assets/custom-assets.md)
- [Hammer Mesh component](../scene/components/reference/hammer-mesh.md) — legacy `.vmap`-based maps
- [Scene Mapping](../scene/scene-mapping/index.md) — building geometry inside s&box
