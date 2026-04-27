---
title: "Editor Widgets"
icon: "⚙️"
sources:
  - engine/Sandbox.Tools/Widgets/BaseItemWidget.cs
  - engine/Sandbox.Tools/Widgets/ToolButton.cs
  - engine/Sandbox.Tools/Widgets/FolderEdit.cs
  - engine/Sandbox.Tools/Widgets/Dialog.cs
  - engine/Sandbox.Tools/Widgets/PopupWidget.cs
created: 2026-04-27
updated: 2026-04-27
---

# Editor Widgets

The `Sandbox.Tools.Widgets` subsystem provides a suite of reusable UI controls built on top of the native Qt binding layer (`Sandbox.Tools.Qt`). These widgets form the building blocks for all custom editor windows, inspectors, and popups.

## Quick Start Example

When building a custom editor dock window, you will construct your layout using these widgets rather than raw Qt primitives.

```csharp
using Editor;
using System.IO;

public class MyCustomToolbar : Widget
{
    public MyCustomToolbar( Widget parent ) : base( parent )
    {
        SetLayout( LayoutMode.LeftToRight );

        var btn = new ToolButton( "Compile", "build", this );
        btn.MouseClick += () => Log.Info( "Compile clicked!" );

        var search = new FolderEdit( this );
        search.PlaceholderText = "Search paths...";
        search.FolderSelected += ( path ) => Log.Info( $"Folder selected: {path}" );
    }
}
```

## Troubleshooting

:::warning
- **Event Leakage:** If you attach a delegate to a widget's event (like `btn.MouseClick += MyMethod`), and `MyMethod` captures a reference to a large subsystem, that subsystem cannot be garbage collected until the widget is destroyed. For complex editor tools, prefer overriding methods or unhooking events in `OnDestroyed()`.
:::

## Related Pages
- [Qt Widget Bindings](qt.md)