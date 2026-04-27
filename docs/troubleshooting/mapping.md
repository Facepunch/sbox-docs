---
title: Mapping & Hammer
icon: "🏔️"
created: 2026-04-25
updated: 2026-04-25
---

# Mapping & Hammer

Maps not loading, Hammer mesh issues, MapInstance behavior, scene mapping problems.

## Map loading

### "My map doesn't show up in my Scene"

`MapInstance` loads the **compiled** artifact, not the raw `.vmap` file. You haven't compiled the map yet.

- **Fix:** in Hammer, press F9 to trigger a full compile.
- **Check:** the path on `MapInstance` points to the compiled file (e.g., `maps/mymap.vmap_c` or its handle).

**Deep dive:** [Hammer](../editor/hammer.md), [Loading Maps](../scene/maps/loading-maps.md).

### "MapInstance.IsLoaded is never true"

The map asset isn't actually present, or it failed to compile silently.

- **Check:** the asset compiles cleanly in Hammer / Mapping tool — look for warnings.
- **Check:** the asset is in your project (not just a sibling addon you don't depend on).

## Hammer Mesh / Scene Mapping

### "Players fall through my custom map geometry"

The Hammer Mesh GameObject has no `Collider`. Mesh renders, doesn't collide.

- **Fix:** add a `ModelCollider` (or any Collider) to the GameObject containing the Hammer Mesh.

**Deep dive:** [Mapping Index](../editor/mapping/index.md), [Hammer Mesh](../scene/components/reference/hammer-mesh.md).

### "I can't edit my mesh vertices"

You're not in Mesh Edit Mode.

- **Fix:** with the Hammer Mesh GameObject selected, press `Space` (or click the mesh-edit icon in the toolbar).

### "Hammer is being deprecated"

Yes — Hammer is on its way out. The replacement is **Scene Mapping** (the in-scene authoring toolset, sometimes called the **Mapping** tool).

- **For new work:** use Scene Mapping. Hammer Mesh components and `.vmap` workflows still work for legacy maps.

**Deep dive:** [Scene Mapping](../scene/scene-mapping/index.md).

### "Hammer entity doesn't spawn at runtime"

The entity wasn't bound to a Component, or the GameData definition is missing fields the engine expected.

- **Check:** the entity class has `[Property]`s that match the Hammer FGD/GameData.
- **Check:** unsupported property types (nested object collections) silently fail to serialize from Hammer.

**Deep dive:** [Map Entities](../scene/maps/map-entities.md), [Tools GameData](../engine-internals/sandbox-tools/gamedata.md).

## Trigger volumes from maps

### "Trigger Events Not Firing on a Hammer Mesh"

`IsTrigger` is on, but the entering object isn't capable of triggering — or collision tags exclude each other.

- **Check:** the entering object has a `Rigidbody` or a Collider that interacts with triggers.
- **Check:** collision tags align (default tag is usually fine).

## Networked maps

### "My networked props in a MapInstance are missing on clients"

Networked objects spawned by the map should only be created by the **host**. If clients also try to spawn, you get desync or duplicates.

- **Fix:** guard the spawn with `if ( Networking.IsHost )`.

**Deep dive:** [Map Networking](../scene/maps/map-networking.md).

## Mapping textures and materials

### "My map shows pink/black checkerboards"

Material path is invalid, the material failed to compile, or the material lives in a mount that wasn't installed.

- **Check:** the material asset compiles (open it in the editor — look for red).
- **Check:** if the texture comes from a mounted game (CSS, HL2), the player has that game installed.

**See also:** [Runtime & Build → checker textures](./runtime-and-build.md).

### "Materials look wrong on ported models from older Source games"

Older engines used specular/gloss; s&box uses PBR roughness/metallic. Auto-conversion isn't perfect.

- **Fix:** open the material, manually adjust roughness/metallic to taste.

**Deep dive:** [Mount Internals](../engine-internals/runtime-layout/mount.md).

## Hammer authoring quirks

### "Hammer.ActiveMap throws — `CHammerApp` not initialized"

You called Hammer-tool APIs before the app was loaded.

- **Fix:** check `Hammer.Open` before reading `Hammer.ActiveMap` or `Hammer.MapAsset`.

**Deep dive:** [Tools Map Editor](../engine-internals/sandbox-tools/mapeditor.md).

### "I have only 4 terrain materials and need more"

Current limit is 4 Terrain Materials. Known limitation, planned to be raised.

- **Workaround:** mask blends, or split the terrain into multiple components for visually distinct zones.

**Deep dive:** [Terrain Materials](../systems/terrain/terrain-materials.md).

## Related Pages

- [Physics](./physics.md)
- [Scene & Components](./scene-and-components.md)
- [Editor](./editor.md)
