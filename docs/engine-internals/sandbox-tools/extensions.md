---
title: "Editor Extensions (Utility Methods)"
icon: "⚙️"
sources:
  - engine/Sandbox.Tools/Extensions/ToolExtensions.cs
  - engine/Sandbox.Tools/Extensions/ActionGraphExtensions.cs
  - engine/Sandbox.Tools/Extensions/PackageExtensions.cs
created: 2026-04-27
updated: 2026-04-27
---

# Editor Extensions (Utility Methods)

The `Sandbox.Tools.Extensions` subsystem provides static extension methods designed exclusively for the tools environment. It bridges standard engine types (like `SceneCamera`, `ActionGraph`, and `Package`) with Qt-specific editor UI constructs (like `Pixmap` and `Widget`).

## Quick Start Example

As a tool developer, you might want to render a scene directly into an editor UI panel (for example, to create a custom asset thumbnail generator or a PiP view). The `ToolExtensions` provide simple wrappers for this:

```csharp
using Sandbox;
using Editor;

public class MyThumbnailGenerator
{
    public void RenderThumbnail( SceneCamera camera, Pixmap targetPixmap )
    {
        // Renders the engine SceneCamera directly into a Qt Pixmap
        if ( camera.RenderToPixmap( targetPixmap ) )
        {
            Log.Info( "Thumbnail generated successfully." );
        }
    }
}
```

## Troubleshooting

:::warning
- **Missing Namespace Reference:** These extensions require `using Sandbox;` and `using Editor;`. Because they extend classes residing in `Sandbox.Engine`, if you forget the `using Sandbox` directive in your tool code, the compiler will fail to find `RenderToPixmap`.
- **Invalid Camera World:** `camera.RenderToPixmap` silently returns `false` if the `camera.World` is null. Ensure the camera is actively participating in an initialized scene before attempting to render a snapshot.
:::

## Related Pages
- [Qt Widget Bindings](qt.md)
