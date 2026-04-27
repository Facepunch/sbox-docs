---
title: Tools Addon
sources:
  - game/addons/tools
created: 2026-04-27
updated: 2026-04-27
---

# Tools Addon

The `tools` addon, located at `game/addons/tools/`, contains the C# logic, Editor Widgets, and UI layouts that make up the s&box Development Editor.

## Educational Value & Extending the Editor

The primary reason to interact with the `tools` addon is to learn how to extend the Editor. If you want to build a custom dockable window, a new type of asset inspector, or a specialized mapping tool, you can look at how the built-in tools are constructed.

### Quick Example: Finding Editor Code

If you want to know how the Scene Tree is populated, you can search the `tools` addon for classes inheriting from `Widget` or `DockWindow`.

```csharp
// Example of the kind of code found in the tools addon:
// Defining a custom Editor Window.
using Editor;

[Dock("Editor", "My Custom Tool", "build")]
public class MyCustomToolWindow : Widget
{
    public MyCustomToolWindow( Widget parent ) : base( parent )
    {
        // Build UI using the Editor UI framework
        var layout = new BoxLayout( BoxLayout.Direction.TopToBottom );
        layout.Add( new Label( "Hello from Editor Code!" ) );
        Layout = layout;
    }
}
```

## Common Patterns

1.  **Widget Trees:** The editor is built using a hierarchy of `Widget` objects, similar to Qt or WinForms, but tightly integrated with s&box.
2.  **Property Sheets:** Many inspector windows dynamically generate their UI by reflecting over C# properties and creating appropriate `PropertyWidget` instances.
3.  **Command System:** The editor makes heavy use of a command pattern for actions (e.g., undo/redo, menu items).

*   **Editor Ticks:** The editor UI ticks constantly. Avoid heavy processing in a widget's update or paint methods, as it will cause the entire editor to lag. Use background tasks for expensive operations like searching or file processing.
*   **Drawing:** When creating custom painting logic for a widget, cache brushes, pens, and layouts where possible.

## Troubleshooting

:::warning Common Gotchas
- **Do not depend on `facepunch.tools` in game code:**
  Code that references the `Editor` namespace or classes from the tools addon **will not compile** when the game is built for the standalone client or server. Editor code must be strictly isolated from gameplay code (usually placed in an `Editor` folder within your project).
- **Accidental modification:**
  Do not edit the files in `game/addons/tools/` to add your custom tools. Instead, write your Editor tools within your own project and ensure they are marked to compile only in Editor mode. Modifying the base tools addon directly will result in your changes being overwritten during engine updates.
:::

## Related Pages
- [Editor Overview](../editor/index.md)
- [Custom Editors](../../editor/editor-widgets.md)

