---
title: "ModelEditor"
icon: "⚙️"
sources:
  - engine/Sandbox.Tools/ModelEditor/ModelDoc.cs
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Tools.ModelEditor

This section covers the internal implementation of the `ModelEditor` (ModelDoc) subsystem within the s&box Editor.

## Architecture

This folder contains the C# source code that handles the editor-side functionality for ModelEditor. It is loaded only when the engine runs in tools mode (`sbox-dev.exe`).

The Model Editor (historically known as ModelDoc) allows developers to view, modify, and compile 3D models. The managed `Sandbox.Tools.ModelEditor` namespace serves as the bridge connecting the native C++ ModelDoc tools to the C# Editor space.

## Core Concepts

The system centers around the static `ModelDoc` class. 

### The `ModelDoc` Bridge

The `Editor.ModelEditor.ModelDoc` class holds a reference to the active `CModelDocEditorApp` instance. It provides simple queries to check if the editor is open and what model is currently being edited.

```csharp
using Editor.ModelEditor;

// Verify if the Model Editor window is currently open
if ( ModelDoc.Open )
{
    // Retrieve the compiled asset of the model currently being edited
    Asset currentModel = ModelDoc.ModelAsset;
    
    if ( currentModel != null )
    {
        Log.Info( $"Currently editing: {currentModel.Path}" );
    }
}
```

### GameData Synchronization

Just like the Map Editor, ModelDoc supports attaching specific engine data or entity definitions to models (such as physics properties, attach points, or custom entity data). When the engine updates game data (e.g., a developer recompiles a C# file that defines a new component or enum), the `ModelDoc` manager catches the `tools.gamedata.refresh` event:

```csharp
[Event( "tools.gamedata.refresh" )]
internal static void RefreshGameData()
{
    if ( App?.IsValid == true )
    {
        // Pushes the updated C# types and GameData into the native ModelDoc UI
        App?.RefreshGameData();
    }
}
```

### Extensibility (Tools Menu)

ModelDoc allows addons and engine tools to inject custom menus via the `OnToolsMenu` hook. The native app passes a `QMenu` object to C#, which is then wrapped in a managed `Menu` class and dispatched via the `modeldoc.menu.tools` Editor Event. This allows C# developers to add custom actions directly to the ModelDoc toolbar without modifying native C++ code.