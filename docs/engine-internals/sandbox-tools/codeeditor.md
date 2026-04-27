---
title: "CodeEditor"
icon: "⚙️"
sources:
  - engine/Sandbox.Tools/CodeEditor/CodeEditor.cs
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Tools.CodeEditor

This section covers the internal implementation of the `CodeEditor` subsystem within the s&box Editor.

## Architecture

This folder contains the C# source code that handles the editor-side functionality for managing external code editors. It is loaded only when the engine runs in tools mode (`sbox-dev.exe`).

Because s&box does not ship with a built-in text editor for C# scripts, it relies on launching external IDEs (like Visual Studio, Visual Studio Code, or Rider). The `Sandbox.Tools.CodeEditor` namespace provides the abstraction layer to discover, select, and launch these external applications.

## Core Concepts

The architecture relies on the `ICodeEditor` interface and the static `CodeEditor` manager class.

### The `ICodeEditor` Interface

Any class that implements `ICodeEditor` is automatically discovered by the engine using the `EditorTypeLibrary`. This allows addons to provide their own code editor integrations without modifying engine source code.

```csharp
public interface ICodeEditor
{
    // Opens a file, optionally scrolling to a specific line/column
    void OpenFile( string path, int? line = null, int? column = null );
    
    // Opens the generated .sln / .slnx for the current project
    void OpenSolution();
    
    // Opens a specific addon directory
    void OpenAddon( Project addon );
    
    // Returns true if the engine detects the editor executable is installed on the user's system
    bool IsInstalled();
}
```

### The CodeEditor Manager

The static `CodeEditor` class is responsible for managing the active editor instance. 

When an action requests to open a file (e.g., double-clicking a `.cs` script in the Asset Browser), the engine calls `CodeEditor.OpenFile(...)`.

1. **Resolution:** If the path is relative, `CodeEditor` searches all active projects (`EditorUtility.Projects.GetAll()`) to resolve the absolute path.
2. **Editor Selection:** It checks the `EditorCookie` (the editor's saved settings) to see which IDE the user has selected. 
3. **Fallback:** If the user's selected editor is not found or not installed, the engine attempts to fall back to `VisualStudio`, then `VisualStudioCode`, and finally any other installed editor it can find via reflection.

## Example: Opening a File

When a user triggers an action that should open source code:

```csharp
using Editor;

// Opens "MyComponent.cs" at line 42 in the user's active external IDE
if ( CodeEditor.CanOpenFile( "Code/MyComponent.cs" ) )
{
    CodeEditor.OpenFile( "Code/MyComponent.cs", line: 42 );
}
```

## Troubleshooting

:::warning No Code Editor Found
If the engine throws a dialog saying "No Code Editor", it means `CodeEditor.Current` failed to resolve any valid, installed IDE. The user must install an IDE like Visual Studio or Visual Studio Code, or the engine's path resolution for the IDE executable (implemented in the specific `ICodeEditor` class) is failing.
:::
