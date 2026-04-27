---
title: "Editor Application Shell"
icon: "⚙️"
sources:
  - engine/Sandbox.Tools/Editor/
created: 2026-04-27
updated: 2026-04-27
---

# Editor Application Shell

The `Sandbox.Tools.Editor` subsystem is the primary container for the s&box Editor's user interface and core application lifecycle. It manages the main window, global menus, and dockable panels that developers interact with daily.

## Quick Start Example

As an engine contributor, you might need to register a new global menu item or hook into the editor's close sequence. This is done by interacting with the `EditorMainWindow` or defining attributes.

```csharp
using Editor;

// Example: Defining a tool that docks into the main editor window
[Dock( "Editor", "My Custom Tool", "build" )]
public class MyCustomTool : Widget
{
    public MyCustomTool( Widget parent ) : base( parent )
    {
        // UI initialization...
    }
}
```

## The Process

When `sbox-dev.exe` launches, the native engine bootstraps the managed C# tools assembly. `EditorMainWindow` is instantiated as the primary view. It queries the `LibrarySystem` (also in this folder) to discover all available tools and attributes, building the UI dynamically. When a user creates a layout or docks a window, the Qt binding layer handles the windowing, but `EditorMainWindow` tracks the state to ensure a clean shutdown.

## Troubleshooting

:::warning
- **Missing Dock Targets:** If you define a `[Dock]` attribute but the window doesn't appear in the Views menu, ensure your class inherits from `Widget` and that the assembly is properly loaded by the tools domain.
- **Console Spam:** The `ConsoleWidget` uses a virtualized list to maintain performance. If you log millions of messages, the underlying log buffer may truncate, but the UI should remain responsive. Ensure you use standard `Log.Info` / `Log.Warning` methods so they route properly.
:::

## Related Pages
- [Qt Widget Bindings](qt.md)
- [Editor API Architecture](../../editor/editor-tools/index.md)
