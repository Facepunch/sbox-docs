---
title: "MapEditor"
icon: "⚙️"
sources:
  - engine/Sandbox.Tools/MapEditor/
updated: 2026-04-25
created: 2026-04-27
---

# Sandbox.Tools.MapEditor

This section covers the internal implementation of the `MapEditor` (Hammer) subsystem within the s&box Editor.

## Architecture

This folder contains the C# source code that handles the editor-side functionality for MapEditor. It is loaded only when the engine runs in tools mode (`sbox-dev.exe`).

The managed `MapEditor` system acts as a bridge between the native Hammer C++ application (`CHammerApp`) and the C# Editor environment. 

## Core Concepts

The architecture relies heavily on the static `Hammer` class inside `Editor.MapEditor`.

### The `Hammer` Class

The `Hammer` static class initializes and manages the connection to the native map editor session. It provides managed access to map documents, selection, and the editor window.

```csharp
using Editor.MapEditor;

// Check if Hammer is open
if ( Hammer.Open )
{
    // Access the currently active MapDocument
    MapDocument activeMap = Hammer.ActiveMap;

    // Get the map's current material asset
    Material currentMaterial = Hammer.CurrentMaterial;
}
```

### MapDocuments and Selection

When Hammer creates or loads a map, it creates a `MapDocument`. You can interact with the current map's scene objects via managed C#:

1. **Object Creation:** C# code can create a new managed `GameObject`, spawn it in the `ActiveMap.World.EditorSession.Scene`, and wrap it in a `MapGameObject`.
2. **Selection:** The native Hammer application fires events to C# which updates the `Selection.Set(...)` utility so the Editor Inspector correctly reflects selected elements in the map.
3. **Prefab Placement:** The managed map editor contains the logic for finding prefabs (`EditorUtility.Prefabs`), instantiating their scenes, breaking them if they aren't templates, and pushing them to the undo stack (`History.MarkUndoPosition`).

## Example: Creating an Object in Hammer

When a user uses the create context menu in the map view, the managed layer handles the logic:

```csharp
using Editor.MapEditor;

internal static void SpawnObjectInMap()
{
    if ( !Hammer.Open ) return;

    // Push the Hammer editor scene scope
    using var scope = Hammer.ActiveMap.World.EditorSession.Scene.Push();
    
    // Create a new generic GameObject
    var go = new GameObject( true, "Object" );

    // Wrap it for the Map Editor
    var mgo = new MapGameObject( Hammer.ActiveMap, go );

    // Select the new object in the Hammer UI
    Selection.Set( mgo );
}
```

## Troubleshooting

:::warning App Validation
Calling properties on the `Hammer` class (like `Hammer.ActiveMap` or `Hammer.MapAsset`) before checking `Hammer.Open` will throw an exception if the `CHammerApp` hasn't been fully initialized or loaded a map.
:::

## Subsystems

### DragDrop (`MapEditor/DragDrop/`)
This subsystem handles the logic for dropping items directly into the map editor's viewport.
*   **`IMapViewDropTarget`:** The interface defining drop targets for the map view.
*   **Implementations:** Classes like `MaterialDropTarget`, `ModelDropTarget`, and `EntityDropTarget` implement the logic to instantiate prefabs, apply materials to blocks, or spawn models when dragged from the Asset Browser into the scene.

### HammerEntities (`MapEditor/HammerEntities/`)
This directory contains managed C# representations of specific entities that exist within the native Hammer editor.
*   These classes (e.g., `PointLightEntity`, `EnvironmentLight`, `PropStatic`, `FuncBrush`) often interface with the `GameData` definitions to ensure properties modified in the Inspector correctly sync back to the C++ document.

### MapDoc Nodes (`MapEditor/MapDoc/Nodes/`)
This acts as the C# object model for the currently loaded map document. It is a hierarchical representation of everything in the map.
*   **`MapNode`:** The base class for all items in the map document hierarchy.
*   **`MapEntity` & `MapGameObject`:** Represent logical entities and Scene GameObject wrappers.
*   **`MapMesh` & `MapGroup`:** Represent geometry (blocks) and grouping folders within the map.

#### Manipulating MapNodes

You can traverse and modify the map document structure by casting `MapNode`s to their specific types:

```csharp
using Editor.MapDoc;

// Finding all prop_physics entities
foreach ( var node in Hammer.ActiveMap.World.Children )
{
    if ( node is MapEntity entity && entity.ClassName == "prop_physics" )
    {
        Log.Info( $"Found a prop: {entity.GetKeyValue( "model" )}" );
        
        // Change the model
        entity.SetKeyValue( "model", "models/props_c17/oildrum001_explosive.mdl" );
    }
}
```

```csharp
using Editor.MapDoc;

// Accessing GameObjects wrapped by Hammer
foreach ( var node in Hammer.ActiveMap.World.Children )
{
    if ( node is MapGameObject mapGo && mapGo.GameObject is not null )
    {
        Log.Info( $"Found wrapped GameObject: {mapGo.GameObject.Name}" );
        mapGo.GameObject.Enabled = false; // Disable it in the scene
    }
}
```

### Tools (`MapEditor/Tools/`)
Managed tools that users can activate within the editor (like the block tool or entity tool).
*   **`BlockTool`, `EntityTool`, `PathTool`:** Define the interaction logic (mouse input, clicking, dragging) for creating geometry or placing entities and paths in the viewport.

### Custom Tool Modes

The map editor allows adding tools by implementing interfaces that are hooked up to native `CTool*` classes via "Glue" static classes.

For example, the Block Tool implements `IBlockTool` and is called natively from `BlockToolGlue`:

```csharp
using Editor.MapEditor;
using Editor.MeshEditor;

public class MyCustomBlockTool : IBlockTool
{
    public PrimitiveBuilder Current { get; set; } = new();
    public bool InProgress { get; set; }
    public string EntityOverride { get; set; }

    public MyCustomBlockTool()
    {
        // Must assign the singleton instance so the native Glue class can find it.
        // This connects your C# tool logic to the native CToolBlock in C++.
        IBlockTool.Instance = this;
    }

    public Widget BuildUI()
    {
        // Return a custom Qt Widget to display tool settings in the active tool properties panel.
        var widget = new Widget();
        widget.Layout = Layout.Column();
        
        var title = new Label( "Custom Block Logic" );
        title.SetStyles( "font-weight: bold; margin-bottom: 8px;" );
        widget.Layout.Add( title );
        
        var orientCheckbox = new Checkbox( "Orient Primitives" );
        orientCheckbox.Value = IBlockTool.OrientPrimitives;
        orientCheckbox.StateChanged += ( state ) => 
        {
            IBlockTool.OrientPrimitives = state;
            NotifyGeometryChanged();
        };
        
        widget.Layout.Add( orientCheckbox );
        return widget;
    }

    // Call this to notify native Hammer to request a geometry redraw based on `Current`.
    public void NotifyGeometryChanged()
    {
        IBlockTool.UpdateTool();
    }
}
```

## Related Components
- **MapViews:** The managed `Hammer` class also processes the screen-space HUD rendering on top of map views via `RenderMapViewHUD`.
- **GameData Refresh:** Whenever entities are updated, the `tools.gamedata.refresh` event invokes `App.RefreshEntitiesGameData()` to synchronize C# node definitions with the native Map Editor.