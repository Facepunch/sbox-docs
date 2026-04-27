---
title: Editor API Architecture
icon: "✏️"
sources:
  - engine/Sandbox.Tools
created: 2026-04-27
updated: 2026-04-27
---

# Editor API Architecture

The Editor API wraps the native Qt GUI framework to provide a comprehensive C# API for building editor tools, custom inspectors, dockable windows, and native tool integrations inside the s&box Editor.

## Quick Start Example

You can easily create a new dockable window by inheriting from `DockWindow` and exposing it via the `[Menu]` attribute:

```csharp
using Sandbox;
using Editor;

public sealed class MyCustomEditorTool
{
    // This attribute places "My Custom Tool" under the "Editor" menu bar.
    [Menu( "Editor", "My Custom Tool" )]
    public static void OpenMyTool()
    {
        var window = new DockWindow();
        window.Title = "My Custom Tool";
        window.Size = new Vector2( 400, 300 );
        
        // Add a basic Qt layout and label
        var layout = new BoxLayout( BoxLayout.Direction.TopToBottom );
        var label = new Label( "Welcome to the Editor API!" );
        layout.Add( label );
        window.Layout = layout;

        window.Show();
    }
}
```

## Common Patterns

1. **Custom Inspectors:** Use the Editor API to create custom `PropertySheet` layouts for your GameObjects or Assets.
2. **Dock Windows:** Build persistent native windows that can be docked into the main editor workspace.
3. **Menu Commands:** Add actions directly to the top menu bar or right-click context menus in the Asset Browser via the `[Menu]` and `[Menu.Asset]` attributes.

## Performance

*   **Native Interop Overheads:** Because the Editor API is wrapping C++ Qt objects, creating and destroying thousands of widgets per frame (like populating a massive list without virtualizing it) will cause severe GC overhead and lock up the editor thread. Use virtualized list views or item models for massive datasets.

*   **Window Lifecycles:** When a user closes a `DockWindow`, it is destroyed by default. If you need it to persist its state when closed and reopened, you must intercept the close event or save its configuration data externally to restore it upon recreation.

## Troubleshooting

:::warning "Window instantly closes or crashes!"
If you instantiate a native Qt widget but do not assign it a parent or immediately call `.Show()`, the garbage collector may collect the C# wrapper, subsequently deleting the underlying C++ pointer. Ensure you hold a strong reference to your active windows or root widgets.
:::

## Related Pages
- [Engine Internals](../../engine-internals/index.md)
