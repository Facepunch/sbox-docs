---
title: "Hammer (Mapping)"
icon: "🗺️"
sources:
  - game/editor/Hammer/Code/
updated: 2026-04-27
created: 2026-04-27
---

# Hammer (Mapping)

:::danger Hammer is being deprecated — use Scene Mapping for new work
Hammer is on its way out. The replacement is **Scene Mapping** — the in-scene authoring toolset (officially called the **Mapping** tool, with `[Title("Mapping")]` on `MeshTool` in the engine source). You build geometry, lighting, and gameplay directly in the scene editor using `MeshComponent` + `PolygonMesh`, save the result as a `.scene` file, and load it at runtime via `MapInstance.MapName = "path/to/level.scene"`.

**Start here:** [Scene Mapping](../scene/scene-mapping/index.md) — full workflow doc covering the Mapping tool, primitive editors, Face/Edge/Vertex tools, modifiers (Bevel, Bridge, Clip, Displacement, EdgeArch, EdgeCut, Mirror), texturing, and runtime loading.

**Why this is the path:** the Mapping tool already ships in the scene editor toolbar. `MapInstance` natively recognizes `.scene` files — its engine docstring is *"Allows you to load a map into the Scene. This can be either a vpk or a scene map"* — and `LoadMapAsync` has a dedicated `.scene` branch. `MeshComponent`/`PolygonMesh` are first-class and engine-supported.

Hammer is **not yet hard-removed** — it still ships, `.vmap` files still work, existing maps don't break. The [`HammerMesh`](../scene/components/reference/hammer-mesh.md) component is the bridge that lets you mix old and new workflows if you need to. Don't start new projects with `.vmap`-first authoring.
:::

Hammer is the built-in `.vmap`-based level editor for s&box, inherited from the Source 2 engine but heavily modernized. It remains supported for legacy `.vmap` projects — generating navigation meshes, calculating baked lighting, and producing compiled `.vpk` levels — but [Scene Mapping](../scene/scene-mapping/index.md) is the recommended path for new work.

## Quick Working Example

You can add a Hammer Mesh component to any GameObject to create custom map geometry right in the scene.

```csharp
using Sandbox;

// While you usually build Hammer meshes in the Editor, you can interact with them in code:
public sealed class HammerMeshController : Component
{
    [Property] public MapInstance MapToLoad { get; set; }

    protected override void OnStart()
    {
        // Load a compiled .vmap file at runtime
        if ( MapToLoad != null )
        {
            Log.Info( $"Loading map: {MapToLoad.MapName}" );
            // Assigning MapName triggers the load in the current engine version
            MapToLoad.MapName = "maps/my_compiled_map.vpk";
        }
    }
}
```

1. **Creating Static Geometry:** Use the block tool and mesh tools to carve out the floors, walls, and ceilings of your level.
2. **Applying Materials:** The material tool allows you to paint textures onto your geometry faces.
3. **Adding Lights:** Place light entities (like `light_omni` or `light_environment`) to illuminate your map.
4. **Compiling:** You must "Compile" the map (F9) to bake the geometry, lighting, and physics collisions into a format the game can load efficiently.

## Performance

*   **Baked Lighting vs Realtime:** Hammer excels at calculating pre-computed (baked) lighting. Relying on baked lighting for your static environments is vastly more performant than using fully dynamic, shadow-casting realtime lights. 
*   **Mesh Complexity:** While the Source 2 engine handles high poly counts well, excessive use of intricate, un-optimized brushwork can bloat the compiled map size and impact rendering performance. Convert complex details into proper 3D models (`.vmdl`) authored in external software (Blender, Maya) rather than building them out of hundreds of Hammer brushes.

*   **Leaks:** If you are coming from Source 1, you might worry about "Leaks" (where the map must be entirely sealed from the void). Source 2 Hammer **does not require the map to be sealed**. You can have geometry floating in the void without causing compile errors.

## Troubleshooting

:::warning "My map doesn't show up in my Scene!"
Ensure you have actually compiled the map in Hammer. The `MapInstance` component loads the compiled artifact, not the raw `.vmap` file. Press F9 in Hammer to trigger a full compile.
:::

:::danger "Players fall through my custom map geometry!"
Ensure you have added a `ModelCollider` (or another appropriate Collider component) to the GameObject containing your `Hammer Mesh`. Without a Collider, the engine only renders the geometry visually and won't simulate physics.
:::

## See Also

* [**Mapping (Legacy Hammer)**](mapping/index.md) — overview of the legacy Hammer flow and how compiled `.vmap` content surfaces in the runtime.
* [**Scene Mapping Shortcuts**](../scene/scene-mapping/shortcuts.md) — keybinds for the in-scene `MeshTool` (different from Hammer's own keybinds, which live in the Hammer application).

## Sample Projects
* [**Sweeper**](../build-games/samples-templates/sweeper.md) - While a 2D game, it demonstrates the minimal use of a Scene without needing a monolithic map.
* [**Walker Map**](../build-games/samples-templates/walker-map.md) - A simple environment built with the legacy `.vmap` workflow.

## Related Pages
- [MapInstance Component](../scene/components/reference/mapinstance.md)
