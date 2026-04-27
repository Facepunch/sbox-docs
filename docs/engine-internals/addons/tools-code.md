---
title: "Tools Addon Code Architecture"
sources:
  - game/addons/tools/Code/Editor/
  - game/addons/tools/Code/CodeEditors/
  - game/addons/tools/Code/Inspectors/
created: 2026-04-27
updated: 2026-04-27
---

# Tools Addon Code Architecture

The Tools addon (`game/addons/tools/Code/`) contains the C# source code for the s&box Development Editor workspace, including the visual node editors, asset inspectors, and editor extensions.

## Quick Working Example

You can add completely custom functionality to the editor by defining classes that inherit from `Widget` and using `[EditorTool]` attributes.

```csharp
using Sandbox;
using Editor;

// Adding a custom tool to the main Tools menu
[EditorTool( "My Custom Inspector", "build", "Opens the custom inspector window." )]
public class MyCustomInspectorTool : EditorTool
{
    public override void OnClick()
    {
        // For demonstration, simply log to the editor console
        Log.Info("Custom Inspector Tool clicked!");
    }
}
```

1. **Custom Inspectors:** Instead of writing boilerplate UI code, the editor uses reflection to automatically generate `PropertySheet` layouts for classes. Custom property editors can be registered to override how specific data types are drawn.
2. **Dock Windows:** Most editor tools inherit from `DockWindow` or `Widget`, allowing them to be freely torn off, snapped, and arranged by the developer.

- **Editor-Only Context:** Code placed in the `tools` addon or referenced within an `Editor` folder is strictly compiled out of the final game build. Never reference `Editor` namespaces from gameplay classes, or your project will fail to publish.

## Troubleshooting

:::warning Common Gotchas
- **Namespace conflicts:** The tools addon uses the `Editor` namespace heavily. Do not confuse `Sandbox.UI` (game UI) with `Editor.UI` (developer tools UI).
- **Infinite loops in painting:** The editor UI is continuously redrawn. Expensive logic inside a `Widget.Paint()` method will cause the entire s&box editor to stutter.
:::

## Related Pages
- [Tools Addon Overview](index.md)
- [Editor Architecture](../../editor/editor-tools/index.md)

