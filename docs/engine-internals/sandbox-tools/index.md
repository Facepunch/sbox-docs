---
title: "Sandbox.Tools"
icon: "🔧"
sources:
  - engine/Sandbox.Tools/
updated: 2026-04-25
created: 2026-04-27
---

# Sandbox.Tools

The `Sandbox.Tools` namespace contains the internal implementation of the entire s&box Editor environment, bridging the native C++ Qt framework with the C# ecosystem.

## Quick Working Example

As an engine developer, if you want to expose a new native Editor window or widget, you do so here:

```csharp
using Editor;

// Conceptual: A built-in engine tool widget.
// Widget, Layout.Column(), Button and Clicked are all in engine/Sandbox.Tools/Qt/.
internal class InternalDebugWidget : Widget
{
    public InternalDebugWidget( Widget parent ) : base( parent )
    {
        WindowTitle = "Engine Debugger";
        Layout = Layout.Column();

        var btn = new Button( "Dump Memory", this );
        btn.Clicked += () => Log.Info( "Memory dumped." );
        Layout.Add( btn );
    }
}
```

## Common Patterns

1. **Signals and Slots:** Following Qt's architecture, events in `Sandbox.Tools` often use C# standard events that internally bind to Qt Signals.
2. **ControlWidgets:** When the Inspector sees a `[Property] public Color MyColor`, it looks up the registered `ControlWidget` for the `Color` type and instantiates it (in this case, a `ColorControlWidget`). Engine developers add new ControlWidgets here to support new variable types.

## Troubleshooting

:::warning
- **"Widget accessed from wrong thread":** The number one crash when writing Editor code. Ensure background tasks dispatch to the UI thread before updating progress bars or logs.
- **"Attempted to read or write protected memory":** Often caused by trying to access a `Widget` after its parent window has been closed and destroyed by Qt. Check `Widget.IsValid`.
:::

## Sample Validation

Because `Sandbox.Tools` requires a graphical context, unit testing is limited. However, engine developers use the internal `Sandbox.Test.Unit/Editor/` namespace to instantiate headless Qt contexts and verify that basic widget layouts and Inspector reflection logic do not crash.

## Subsystems

These sub-pages explore specific parts of the internal editor implementation.

* [**Animgraph**](animgraph.md): Internal bindings and UI for the animation graph editor.
* [**CodeEditor**](codeeditor.md): Embedded code editor functionality for internal scripting tools.
* [**ControlWidget**](controlwidget.md): The base classes and implementations for Inspector property fields.
* [**Debug**](debug.md): Editor-only debugging overlays and windows.
* [**Editor**](editor.md): Core editor initialization, library management, and shortcuts.
* [**Events**](events.md): The internal event bus for the editor layer.
* [**Extensions**](extensions.md): C# extension methods used throughout the editor codebase.
* [**GameData**](gamedata.md): Parsers and models for FGDs (game data files used by Hammer).
* [**Input**](input.md): Editor-specific input handling (bypassing the game input system).
* [**Inspector**](inspector.md): The dynamic property grid generator powered by `Sandbox.Reflection`.
* [**MapEditor**](mapeditor.md): The C# integration layer for the Hammer map editor.
* [**MeshEditor**](mesheditor.md): Tools for editing raw meshes in the editor.
* [**ModelEditor**](modeleditor.md): The C# integration for the model viewer and editor.
* [**Qt**](qt.md): The massive C# wrapper around the native Qt C++ framework.
* [**Scene**](scene.md): Editor-specific viewports, gizmos, and scene manipulation tools.
* [**Utility**](utility.md): Helpers for standalone publishing and other generic editor tasks.
* [**Widgets**](widgets.md): The standard library of UI controls (Buttons, Trees, Dialogs) used to build the editor.

## Related Pages
- [Editor API Architecture](../../editor/editor-tools/index.md)
