---
title: "Editor Shortcuts"
icon: "⌨️"
created: 2025-02-04
updated: 2025-02-04
sources:
  - engine/Sandbox.Tools/Editor/ShortcutAttribute.cs
  - engine/Sandbox.Tools/EditorShortcuts.cs
---

# Editor Shortcuts

The `[Shortcut]` attribute allows you to bind specific key combinations to trigger methods within Editor Tools or Widgets.

## Quick Working Example

You can add the `[Shortcut]` attribute to any static method to execute logic when a key combination is pressed.

```csharp
using Editor;

public static class MyEditorTools 
{
	[Shortcut("scene.toggle-gizmos", "SHIFT+G")]
	public static void ToggleGizmos()
	{
		// Logic to toggle gizmos here...
		Log.Info("Toggled gizmos!");
	}
}
```

### Creating a Widget Shortcut

If you want a shortcut to apply only when a specific Editor `Widget` is in focus, you can provide the target widget type.

```csharp
using Editor;

public class MyCustomWidget : Widget
{
	// This shortcut will only fire if a SceneViewportWidget is currently in focus
	[Shortcut("mesh.merge", "M", typeof(SceneViewportWidget), ShortcutType.Widget)]
	private void Merge()
	{
		// Merge logic here...
		Log.Info("Merged meshes!");
	}
}
```

## Configuration

The `[Shortcut]` attribute accepts several parameters to configure when and how it executes.

| Parameter | Type | Description |
|:---|:---|:---|
| `identifier` | `string` | A unique string identifier for the shortcut (e.g., `"scene.toggle-gizmos"`). |
| `keyBind` | `string` | The default key combination (e.g., `"SHIFT+G"`, `"M"`). |
| `targetOverride` | `Type` | (Optional) The specific type of Widget that must be in focus to trigger the shortcut. If omitted, it assumes the class the function is defined in is the target type. |
| `type` | `ShortcutType` | (Optional) The scope of the shortcut. Defaults to `ShortcutType.Widget`. |

### Shortcut Types

| `ShortcutType` | Description |
|:---|:---|
| `Widget` | The specific Widget must be in focus. |
| `Window` | A Window containing the Widget must be in focus. |
| `Application` | The Application as a whole must be in focus. |

## Troubleshooting

:::warning Shortcut Not Firing
If your `ShortcutType.Widget` shortcut isn't executing when you press the keybind, ensure that the target widget is actually in focus. If the user has clicked away to the Inspector or Console, the widget shortcut will not trigger.
:::

## Related Pages

* [Editor Widgets](editor-widgets.md)
* [Editor Apps](editor-apps.md)
