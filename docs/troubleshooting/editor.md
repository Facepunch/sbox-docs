---
title: Editor & Inspector
icon: "🎛️"
created: 2026-04-25
updated: 2026-04-25
---

# Editor & Inspector

Components missing from menus, properties not visible, custom editor tools not appearing, gizmos not drawing.

## Component visibility in editor

### "My component doesn't show up in the Add Component menu"

The C# project has a compilation error somewhere — the type registry doesn't load with red errors present.

- **Fix:** clear all compile errors in the Editor console.
- **Check:** class is `public` and inherits from `Component`.
- **Check:** the file is saved to disk.

**See also:** [Hotload & Compile](./hotload-and-compile.md).

### "My [Property] isn't showing up in the Inspector"

- **Check:** the property is `public` (or has `[Property]` explicitly).
- **Check:** the type is supported. Custom classes need `[ConVar]`-style serialization or be marked appropriately.
- **Check:** Inspector is showing the right component — sometimes a stale Inspector view needs a click elsewhere and back.

**Deep dive:** [Property Attributes](../editor/property-attributes.md).

### "Inspector is empty / not showing my GameObject"

`EditorScene.Selection` may be null if no scene is open or all tabs are closed.

- **Fix:** open a scene first, then select the GameObject.

**Deep dive:** [Tools Scene](../engine-internals/sandbox-tools/scene.md).

### "My Hammer property is missing even though it's in the Inspector"

Hammer's native parser doesn't accept all the types the C# Inspector does. Nested object collections silently drop.

- **Fix:** flatten the Hammer-exposed property to a primitive or a supported simple type. Keep complex data on a separate `[Property]` not exposed to Hammer.

**Deep dive:** [Tools GameData](../engine-internals/sandbox-tools/gamedata.md).

## Custom editor tools

### "My EditorTool doesn't appear in the toolbar"

`EditorTool` classes only load from **Editor Projects**. Putting them in a regular Game Project does nothing.

- **Fix:** create or use an Editor project (`Type` is set to a tooling project type), and place `[EditorTool]` classes there.
- **Check:** the class has the `[EditorTool]` attribute and inherits from `EditorTool`.

**Deep dive:** [Editor Tools](../editor/editor-tools/index.md).

### "My App isn't showing up in the menu"

`Editor.Window` subclass not loaded due to compile error or wrong base class.

- **Fix:** inherit from `Editor.Window` and make sure it's `public`.
- **Fix:** clear compile errors.

**Deep dive:** [Editor Apps](../editor/editor-apps.md).

### "My editor window crashes the moment I open it"

You modified `Window.Layout` directly. `Window` exposes `Canvas` instead — modify `Canvas.Layout`.

- **Fix:** assign a new `Widget` to `Canvas`, then modify `Canvas.Layout`.

### "My editor window instantly closes or crashes"

You created a Qt widget without parenting it or calling `.Show()`. The C# wrapper gets garbage-collected, deleting the underlying C++ pointer.

- **Fix:** parent the widget, or call `.Show()` immediately, or hold a reference.

**Deep dive:** [Custom Tools Architecture](../editor/editor-tools/index.md), [Tools Qt](../engine-internals/sandbox-tools/qt.md).

### "My Overlay UI stays on screen forever"

You created `WidgetWindow` (or similar) and didn't clean up.

- **Fix:** in your tool's `OnDisabled`, call `AddOverlay`'s teardown or destroy/null the widget.

## Gizmos and editor rendering

### "My gizmos aren't drawing"

- **Check:** the Component is selected (or has `[EditorHandle]` to always draw).
- **Check:** the gizmo layer is enabled in the viewport toolbar.

**Deep dive:** [Engine Editor Internals](../engine-internals/sandbox-engine/editor.md).

### "Gizmo handles don't respond to clicks"

The handle's hit-test region is too small or you're drawing it in the wrong space.

- **Fix:** check the handle uses the correct `Gizmo.Hitbox.*` API and the hitbox volume is reasonable.

## Shortcuts

### "My ShortcutType.Widget shortcut isn't firing"

The target widget must have focus. If the user clicked the Inspector, the widget shortcut won't fire.

- **Fix:** use `ShortcutType.Window` or `ShortcutType.Application` if you need it to fire regardless of focus.

**Deep dive:** [Editor Shortcuts](../editor/editor-shortcuts.md).

## Threading and crashes

### "Widget accessed from wrong thread"

Qt is strictly single-threaded. You touched a Widget from a `Task` or background thread.

- **Fix:** marshal back to the UI thread before mutating widgets — use `MainThread.Invoke(...)` or your tool's dispatcher.

### "Widget Object was destroyed"

You hold a C# reference to a widget the user closed. Accessing it throws.

- **Fix:** check `Widget.IsValid` before use, or null out the reference when the widget closes.

## Undo and editor scenes

### "My Undo block left state half-applied"

An exception inside `using var undo = new UndoBlock( "Move" )` corrupts the scope.

- **Fix:** wrap the body in `try`/`catch`, or ensure inner code can't throw with the state half-modified.

**Deep dive:** [Undo System](../editor/undo-system.md), [Tools Utility](../engine-internals/sandbox-tools/utility.md).

## Solution generation

### "Visual Studio: 'Sandbox' could not be found"

The `.sln` is stale or the s&box install moved.

- **Fix:** right-click `.sbproj` in s&box → Generate Solution. Delete the old `.sln`/`.csproj` first if needed.

**Deep dive:** [Sandbox.SolutionGenerator](../engine-internals/sandbox-solutiongenerator.md).

## ActionGraph in editor

### "My ActionGraph doesn't do anything when I play"

Your nodes aren't connected to the **white signal wire** from the Root node. Action nodes need an execution signal to run.

- **Fix:** drag from the Root's white output to your first Action node's white input, daisy-chain the rest.

**Deep dive:** [Intro to ActionGraphs](../systems/actiongraph/intro-to-actiongraphs.md).

### "Custom Node not appearing in ActionGraph menu"

The method needs to be `public static`. Instance methods aren't supported as standalone nodes.

- **Fix:** make the method `public static`. Use `[ActionGraphNode]` to expose with a custom name.

**Deep dive:** [C# Method Nodes](../systems/actiongraph/custom-nodes/c-method-nodes.md).

### "Cannot Create Custom Node button is missing"

The selection can't be cleanly isolated — it includes the root, or has bad data flow.

- **Fix:** don't include the root in your selection. Make sure inputs/outputs to the selection are clear.

**Deep dive:** [Action Resources](../systems/actiongraph/custom-nodes/action-resources.md).

## Code Editor integration

### "Editor says No Code Editor Found"

`CodeEditor.Current` couldn't resolve any installed IDE.

- **Fix:** install Visual Studio, VS Code, or Rider, then in Editor Settings → Code Editor pick the one you installed.

**Deep dive:** [Tools CodeEditor](../engine-internals/sandbox-tools/codeeditor.md).

## Related Pages

- [Hotload & Compile](./hotload-and-compile.md)
- [UI / Razor](./ui-razor.md)
- [Mapping](./mapping.md)
