---
title: "Editor Events System"
icon: "⚙️"
sources:
  - engine/Sandbox.Tools/Events/EditorEvent.cs
  - engine/Sandbox.Tools/Events/EditorEvent.MapEditor.cs
created: 2026-04-27
updated: 2026-04-27
---

# Editor Events System

The `Sandbox.Tools.Events` subsystem manages the global event bus within the s&box Editor environment. It allows distinct editor tools, plugins, and native subsystems to broadcast and listen for state changes without maintaining direct references to one another.

## Quick Start Example

As an engine contributor or custom tool developer, you can hook into editor-wide events (like the per-frame update loop or a hotload trigger) by using the `EditorEvent` attributes on static methods.

```csharp
using Editor;
using Sandbox;

public static class MyCustomToolHooks
{
    // Executes every frame within the editor's update loop
    [EditorEvent.Frame]
    public static void OnEditorFrame()
    {
        // Custom editor update logic
    }

    // Triggers automatically whenever the C# codebase is recompiled and hotloaded
    [EditorEvent.Hotload]
    public static void OnHotloaded()
    {
        Log.Info( "Editor detected a hotload! Refreshing UI..." );
    }
}
```

## Common Tool Events

Beyond global hooks, the system exposes specific events for deep tool integration:

### MapEditor (Hammer) Events
If you are extending the Map Editor, you can hook into user interactions:
- `[EditorEvent.MapEditor.SelectionChanged]` - Fires when the user selects or deselects nodes/entities in the Hammer viewport.
- `[EditorEvent.MapEditor.MapViewContextMenu]` - Fires when a user right-clicks in the Hammer 3D/2D views. The callback receives an `Editor.Menu` object, allowing custom plugins to dynamically inject new context menu options.

## Troubleshooting

:::warning
- **Ghost Events After Hotload:** If your custom tool circumvents the `EditorEvent` attributes and registers delegates manually (e.g., using standard C# `event Action`), you *must* unregister them in a destructor or cleanup routine. Otherwise, every time you press save, a new instance of your listener is attached alongside the old one, causing duplicate executions and memory leaks. The `EditorEvent` attributes avoid this by managing assembly cleanup automatically.
:::

## Related Pages
- [Global Event System](../../systems/events.md)
