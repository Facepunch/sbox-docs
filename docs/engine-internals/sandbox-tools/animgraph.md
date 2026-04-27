---
title: "Animgraph"
icon: "⚙️"
sources:
  - engine/Sandbox.Tools/Animgraph/Animgraph.cs
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Tools.Animgraph

This section covers the internal implementation of the `Animgraph` subsystem within the s&box Editor.

## Architecture

This folder contains the C# source code that handles the editor-side functionality for Animgraph. It is loaded only when the engine runs in tools mode (`sbox-dev.exe`).

The Animgraph namespace bridges the native C++ Animgraph tools with C# Editor widgets. The primary integration point is the `ModelPicker`, which allows developers to select and preview models directly within the native Animgraph preview dock.

## Quick Working Example

The `ModelPicker` wraps the native `CQAnimGraphPreviewDockWidget`. It uses the `ResourceControlWidget` from the editor's ControlWidget library to provide a consistent UI for picking model resources.

```csharp
using Native;
using Editor;

// The ModelPicker interfaces with the native Animgraph preview dock
internal static class Animgraph
{
    private class ModelPicker : Widget
    {
        private NativeAnimgraph.CQAnimGraphPreviewDockWidget parent;

        public Model Model
        {
            get
            {
                var path = parent.GetPreviewModel();
                return string.IsNullOrWhiteSpace( path ) ? null : Model.Load( path );
            }
            set => parent.SetPreviewModel( value?.ResourcePath );
        }
        
        // ... layout setup utilizing ResourceControlWidget
    }
}
```

## Common Patterns

1.  **Native Wrapping**: The C# code creates managed `Widget` objects that contain a reference to a native Qt widget (in this case, `CQAnimGraphPreviewDockWidget`). It uses standard C# properties to get and set values via the native methods (`GetPreviewModel` and `SetPreviewModel`).
2.  **ControlWidget Reuse**: Rather than building a custom UI to select models, the `ModelPicker` instantiates the `ResourceControlWidget` via reflection (`EditorTypeLibrary.GetType<ControlWidget>( "ResourceControlWidget" )`), ensuring visual and behavioral consistency with the rest of the Editor Inspector.
