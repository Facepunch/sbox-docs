---
title: "Inspector"
icon: "⚙️"
sources:
  - engine/Sandbox.Tools/Inspector/
  - engine/Sandbox.Tools/Inspector/CanEditAttribute.cs
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Tools.Inspector

This section covers the internal implementation of the `Inspector` subsystem within the s&box Editor.

## Architecture

This folder contains the C# source code that handles the editor-side functionality for generating UI widgets in the Editor Inspector pane. It is loaded only when the engine runs in tools mode (`sbox-dev.exe`).

The primary goal of the Inspector subsystem is to provide a reflection-driven UI builder. When a user clicks a C# `GameObject` or `Component` in the Editor, the Inspector reads its properties and automatically creates the correct UI fields (widgets) to display and edit those values.

## Core Concepts

The architecture relies on high-level property mapping through `CanEditAttribute` and specifically for inspector property sheets through `ControlWidget` generation powered by `CustomEditorAttribute`.

### 1. `InspectorWidget`

Any UI class designed to inspect an object inherits from `InspectorWidget`. This abstract base class holds a reference to a `SerializedObject`, which represents the target data being edited.

```csharp
using Editor;

// Example of a custom inspector widget
public class MyCustomInspectorWidget : InspectorWidget
{
    public MyCustomInspectorWidget( SerializedObject so ) : base( so )
    {
        // Setup UI layout based on the SerializedObject properties
    }
}
```

The static method `InspectorWidget.Create( SerializedObject obj )` is the entry point for generating a full Inspector panel. It queries the `EditorTypeLibrary` to find a matching widget type registered via `[InspectorAttribute]`.

### 2. Field Rendering & `CustomEditorAttribute`

When the Inspector generates an editor for a specific property (like a `float`, `Vector3`, or an `enum`), it relies on the `ControlWidget` subsystem. The correct widget is identified via `[CustomEditorAttribute]`.

```csharp
// Conceptual example inside the engine:
[CustomEditor( typeof( Vector3 ) )]
public class Vector3ControlWidget : ControlWidget
{
    // ... UI implementation for editing an X, Y, Z float field
}
```

When creating a widget for a `SerializedProperty`, `ControlWidget.Create( SerializedProperty property )` checks the `CustomEditorAttribute` against the property's type. It uses a scoring system to pick the best match (e.g., explicit named editors vs base type matches).

### 3. High-level Mapping with `CanEditAttribute`

While `CustomEditorAttribute` is tightly coupled to `ControlWidget` for property fields, `CanEditAttribute` serves a broader role. It is used as a generic mapping tool to associate UI widgets with specific types or assets across the editor (e.g., in the Asset Browser or specialized popups).

```csharp
// Using CanEditAttribute to map an editor to an asset extension
[CanEdit( "asset:vsndstck" )]
public class SoundStackEditor : Widget, IAssetEditor
{
    // ... UI for editing sound stacks
}
```

When a system needs to instantiate an editor generically, it calls `CanEditAttribute.CreateEditorFor()`. This evaluates a highly-weighted score check:

1. **Explicit Definitions:** If a property has `[Editor("CustomName")]`, it searches for a `[CanEdit("CustomName")]` widget first (highest score).
2. **Type Matching:** It checks if any widget explicitly targets the exact type.
3. **Generic & Base Types:** It falls back to generic constraints or base classes if an exact match isn't found.
4. **Arrays/Lists:** If the type is an array or an `IReadOnlyList`, it dynamically constructs a `MultiSerializedObject` to edit multiple elements via common base types.

## Example: Creating an Editor Field dynamically

If an engine developer needs to instantiate an editing widget for an object generically:

```csharp
using System.Reflection;
using Editor;

public Widget GenerateGenericEditor( object myObject )
{
    // Automatically creates the best matching widget for the object
    Widget editorField = CanEditAttribute.CreateEditorForObject( myObject );
    return editorField;
}
```

## Related Components
- **ControlWidgets:** The actual implementations of the fields (like sliders or color pickers) live inside `engine/Sandbox.Tools/ControlWidget/` and rely on `CustomEditorAttribute` to bind to user code.
- [ControlWidget Subsystem](controlwidget.md)