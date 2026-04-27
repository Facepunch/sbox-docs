---
title: "Editor"
icon: "🔩"
sources:
  - engine/Sandbox.Tools/Utility/Utility.cs
updated: 2026-04-25
created: 2026-04-27
---

# Editor

The s&box Editor is a comprehensive suite of tools built for creating games, editing assets, and managing your projects directly within the engine.

![s&box editor](https://files.facepunch.com/matt/1b2211b1/sbox-dev_FoZ5NNZQTi.jpg)
*The s&box Editor Interface*

## Quick Start
```csharp
using Editor;

// Example of opening a specific path from Editor code
EditorUtility.OpenFolder( "/logs/" );
```

## Common Patterns

1. **Custom Inspectors:** Use the `[Property]` attribute on your Components to automatically generate UI in the Editor's Inspector. You can further customize this with attributes like `[Range]` or `[Title]`.
2. **Editor Windows:** Create entirely new floating windows by inheriting from `Editor.Window` or `Widget`.
3. **Editor Only Code:** Place your editor extensions in specific folders or projects that are marked for Editor-only compilation so they don't get shipped to players.

## Navigation Hub

### Customizing the Editor
* [**Editor Widgets**](editor-widgets.md) - Reusable UI components for building custom editor tools.
* [**Custom Editors**](custom-editors.md) - How to write your own specialized editor windows.
* [**Editor Apps**](editor-apps.md) - Creating full applications within the editor environment.
* [**Editor Tools**](editor-tools/) - Specialized tools created specifically for scene editing.

### Built-in Tools
* [**Scene Mapping**](../scene/scene-mapping/index.md) — the in-scene **Mapping** tool (`MeshTool`) for authoring levels with [`MeshComponent`](../scene/components/reference/meshcomponent.md) + [`PolygonMesh`](../scene/components/reference/polygonmesh.md). The recommended path for new projects.
* [**Hammer (legacy)**](hammer.md) — the older `.vmap`-based level editor. Deprecated for new work; see the deprecation note on its page. The legacy [Mapping reference](mapping/index.md) covers Hammer-specific shortcuts and workflows.
* [**Action Graph**](action-graph.md) — visual scripting.
* [**DooEditor**](doo-editor.md) — block-based logic editor.
* [**Movie Maker**](movie-maker.md) — cinematic / animation authoring.
* [**Shader Graph**](shader-graph.md) — node-based shader authoring.
* [**Model Editor**](model-editor.md) — inspecting and modifying 3D models.
* [**Editor Shortcuts**](editor-shortcuts.md) — keyboard shortcuts for a faster workflow.

## Troubleshooting
:::warning Common Gotchas
- **"My custom editor tool isn't showing up!"**
  Ensure your class is decorated with the correct editor attributes (e.g., `[EditorTool]`) and is located in a folder or project that the editor specifically compiles for the Editor context. Editor code will not run in the game client.
- **"Using 'Editor' namespace causes build errors in my game."**
  You cannot use the `Editor` namespace or classes inside your standard `Component` code that runs in the game. You must isolate Editor code.
- **Editor crashing on hotload:**
  If you write unstable custom editor code (like infinite loops in UI drawing), it can crash the entire s&box editor. Always test UI logic carefully.
:::

## Performance
- **Editor Update Loops:** If your custom Editor Window or Tool uses a tick function (`[EditorEvent.Frame]`), ensure it doesn't perform heavy work like loading large assets or running deep file system scans synchronously, as this will drop the frame rate of the entire Editor and game preview.

- **Game Mode vs Editor Mode:** Be careful writing Editor scripts that assume the `Scene` is actively playing. Some editor events run while the scene is in "Edit" mode (paused), so logic relying on `Time.Delta` or physics updates may behave unexpectedly.

## Sample Projects
- [**Sweeper Sample**](../build-games/samples-templates/sweeper.md) - Open this project in the Editor to explore how a full game utilizes custom assets and structured scene files.

## Related Pages
* [Getting Started](../getting-started/index.md)
* [Code](../code/index.md)
* [Maps (runtime loading)](../scene/maps/index.md) — how the levels you author here actually get loaded into a scene.
