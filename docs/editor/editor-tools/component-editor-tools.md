---
title: "Component Editor Tools"
icon: "📚"
created: 2023-12-29
updated: 2023-12-29
sources:
  - game/addons/tools/Code/Scene/Tools/EditorTool.cs
  - game/addons/tools/Code/Scene/Tools/ComponentEditorTool.cs
  - game/addons/tools/Code/Scene/Tools/EditorToolManager.cs
---

# Component Editor Tools

Component Editor Tools work like regular [Editor Tools](index.md), but they activate automatically when a specific [Component](../../scene/components/index.md) is selected and deactivate when the selection no longer contains that component. They're how you draw scene-view UI, gizmos, and overlays for a particular component without polluting any other selection state.

![The camera preview is created using a Component EditorTool](./images/the-camera-preview-is-created-using-a-component-editortool.png)

The camera preview overlay you see when you select a `GameObject` with a `CameraComponent` is built this way.

## Defining a tool

Inherit from `EditorTool<T>` where `T` is the component type you want to target. The generic constraint is `where T : Component`:

```csharp
using Editor;
using Sandbox;

public class MyEditorTool : EditorTool<MyComponent>
{
    public override void OnEnabled()
    {
        // Tool just became active — set up overlays, register hotkeys
    }

    public override void OnUpdate()
    {
        // Called every frame while the tool is active
    }

    public override void OnDisabled()
    {
        // Tool is being torn down — clean up
    }

    public override void OnSelectionChanged()
    {
        // The selection contents changed but the tool is still active
        var target = GetSelectedComponent<MyComponent>();
    }
}
```

`EditorTool<T>` is just `public class EditorTool<T> : EditorTool where T : Component { }` — a thin generic shim over the non-generic `EditorTool` base. Use the generic form for component-specific tools; use the non-generic `EditorTool` directly when you're building a tool that isn't tied to a particular component (custom mapping subtools, terrain tools, gizmo modes).

## Lifecycle

| Method | When it fires |
|---|---|
| `OnEnabled()` | When the tool is first registered (the targeted component appears in the selection). |
| `OnUpdate()` | Every frame while the tool is active. Wrapped in a try/catch — exceptions are logged, not propagated. |
| `OnSelectionChanged()` | When the editor selection changes but the tool stays alive (e.g. another GameObject with the same component type was added to the selection). |
| `OnDisabled()` | When the tool is being torn down. |
| `Dispose()` | Final cleanup — destroys overlay widgets and disposes sub-tools. Calls `OnDisabled` for you. |

By default the tool is auto-destroyed when the selection no longer contains the target component. Override `ShouldKeepActive()` to keep it alive past that point:

```csharp
public override bool ShouldKeepActive() => true;
```

## What the base class gives you

Useful properties available on every `EditorTool`:

| Property | Type | Purpose |
|---|---|---|
| `Scene` | `Scene` | The scene the editor session is showing. |
| `Selection` | `SelectionSystem` | Current selection, queryable with LINQ. |
| `Camera` | `CameraComponent` | The viewport camera (set by `Frame`). |
| `SceneOverlay` | `Widget` | The active scene-overlay widget — parent for any overlay UI you add. |
| `Trace` | `SceneTrace` | Pre-built trace from the current mouse cursor through the scene. |
| `MeshTrace` | `SceneTrace` | Like `Trace` but ignores physics and traces against rendered meshes — useful for picking visual-only geometry. |
| `Tools` | `IEnumerable<EditorTool>` | Sub-tools (see "Sub-tools" below). |
| `CurrentTool` | `EditorTool` | The currently-active sub-tool. |

Useful methods:

```csharp
// Pull the selected component out of the selection (returns null if none).
protected T GetSelectedComponent<T>() where T : Component;

// Add a widget to the scene viewport overlay, aligned to a corner.
public void AddOverlay( Widget widget, TextFlag align = TextFlag.RightTop, Vector2 offset = default );

// Duplicate the current selection (Ctrl+D-equivalent).
protected void DuplicateSelection();

// Get the current selection as a SerializedObject (single or multi-selection).
public SerializedObject GetSerializedSelection();
```

## Adding UI

A tool can contribute widgets to the editor's chrome by overriding any of these:

```csharp
public override Widget CreateToolSidebar();      // Left toolbar — main tool UI
public override Widget CreateToolbarWidget();    // Top viewport toolbar — option strip
public override Widget CreateToolFooter();       // Bottom of the tool list
public override Widget CreateShortcutsWidget();  // Shortcut-key surface (parents inherit)
```

For overlays inside the scene viewport itself (info readouts, transform handles, status badges), use `AddOverlay` from `OnEnabled`:

```csharp
public override void OnEnabled()
{
    var label = new Label( "Editing my component" );
    AddOverlay( label, TextFlag.LeftTop, new Vector2( 12, 12 ) );
}
```

Overlays added this way are tracked and destroyed for you in `Dispose()`.

## Box and lasso selection

Most tools don't need this, but if yours involves selecting things in the viewport:

```csharp
public override bool HasBoxSelectionMode()   => true;
public override bool HasLassoSelectionMode() => true;

protected override void OnBoxSelect( Frustum frustum, Rect screenRect, bool isFinal ) { }
protected override void OnLassoSelect( List<Vector2> lassoPoints, bool isFinal ) { }
```

The base class handles drag-detection and rendering of the selection rectangle/lasso for you; you just react to the selection result. Use the static helper `IsPointInLasso( point, lassoPoints )` to test individual screen points against a lasso polygon.

## Sub-tools

Override `GetSubtools()` to nest tools inside your tool — useful for mode-switching (e.g. translate / rotate / scale within the same parent):

```csharp
public override IEnumerable<EditorTool> GetSubtools()
{
    yield return new TranslateSubTool();
    yield return new RotateSubTool();
    yield return new ScaleSubTool();
}
```

The first one returned becomes `CurrentTool`. Switch between them by assigning to `CurrentTool` — the previous sub-tool's `OnDisabled` and the new one's `OnEnabled` fire automatically.

## Common pitfalls

:::warning OnUpdate exceptions are swallowed
The `Frame` method wraps `OnUpdate` in a try/catch and logs the exception as a warning — it doesn't crash the editor. If your overlay isn't updating, check the editor console for warnings before assuming the method isn't being called.
:::

:::warning GetSelectedComponent returns null when nothing is selected
The auto-deactivation kicks in when the *target component type* is no longer selected — but `OnSelectionChanged` can fire with a transient empty selection. Always null-check the result.
:::

## See also

- [Editor Tools](index.md) — the non-component-specific tool flavour.
- [Component Lifecycle](../../scene/components/component-lifecycle.md) — what your target component is doing while your tool watches it.
- [Editor Project](../editor-project.md) — how to set up an editor-only assembly so your tool lives outside the sandboxed game code.
