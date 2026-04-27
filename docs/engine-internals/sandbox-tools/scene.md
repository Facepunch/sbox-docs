---
title: "Editor Scene Management"
icon: "⚙️"
sources:
  - engine/Sandbox.Tools/Scene/EditorScene.cs
  - engine/Sandbox.Tools/Scene/SceneExtensions.cs
  - engine/Sandbox.System/Utility/SelectionSystem.cs
created: 2026-04-27
updated: 2026-04-27
---

# Editor Scene Management

The `Sandbox.Tools.Scene` subsystem orchestrates the lifecycle, state, and serialization of `Scene` objects specifically within the context of the s&box Editor. It manages active sessions, undo/redo states, and transitions between Edit Mode and Play Mode.

## Quick Start Example

As a tool developer, you often need to interact with the currently active scene in the editor (e.g., to spawn a preview object or alter the global selection). You do this via the `EditorScene` static class:

```csharp
using Editor;

public class MySceneTool
{
    public void SelectAllChildren( GameObject root )
    {
        // Add children to the global editor selection pool
        foreach ( var child in root.Children )
        {
            EditorScene.Selection.Add( child );
        }
    }

    public void TogglePlayMode()
    {
        // Toggle whether hitting 'Play' launches the active scene or the project default
        EditorScene.PlayMode = !EditorScene.PlayMode;
    }
}
```

## The Process: Edit vs Play Mode

When a user clicks the "Play" button, `EditorScene` handles the state transition:
1. It validates the active session and checks for compilation errors.
2. It serializes the current `Scene` into a temporary JSON state.
3. It destroys the editor instance of the scene and constructs a new runtime `Scene` instance.
4. When "Stop" is clicked, it destroys the runtime scene, deserializes the JSON state back into a new editor `Scene`, and restores the `SelectionSystem` state.

## Troubleshooting

:::warning
- **Null Selection:** `EditorScene.Selection` can be null if no scene is actively open (e.g., when the editor first launches or all tabs are closed). Always null-check before appending to the selection.
- **PlayMode Desync:** If a component fails to serialize properly (due to missing `[Property]` tags or unsupported types), the state restored after stopping Play Mode will be incomplete.
:::

## Related Pages
- [Scene System Architecture](../../systems/scene-system/architecture.md)
