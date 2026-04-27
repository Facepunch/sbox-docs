---
title: Sandbox.Engine.Editor
icon: "✏️"
sources:
  - engine/Sandbox.Engine/Editor/Gizmos/Gizmo.cs
  - engine/Sandbox.Engine/Editor/Hammer/HammerAttributes.cs
updated: 2026-04-25
created: 2026-04-27
---

# Sandbox.Engine.Editor

The `engine/Sandbox.Engine/Editor/` directory contains internal, highly-privileged APIs that integrate the managed C# environment with the s&box native Editor suite. It handles critical authoring components like Gizmo rendering and Hammer map compilation logic.

## Quick Working Example

If you are developing complex editor tools or custom components, you will likely interact with the Gizmo system to draw handles and debug lines in the scene viewport:

```csharp
using Sandbox;

public sealed class CustomRadiusComponent : Component
{
    [Property] public float Radius { get; set; } = 100f;

    // Component.DrawGizmos is the protected virtual called per frame
    // when the gizmo system traverses the scene.
    // See engine/Sandbox.Engine/Scene/Components/Component.Gizmos.cs.
    protected override void DrawGizmos()
    {
        Gizmo.Draw.Color = Color.Yellow;
        Gizmo.Draw.LineSphere( Vector3.Zero, Radius );

        // Gizmo.Control.Position is the canonical position-handle helper.
        // See engine/Sandbox.Engine/Editor/Gizmos/Control/Control.cs.
        if ( Gizmo.Control.Position( "radius_handle", Vector3.Up * Radius, out var newPosition ) )
        {
            Radius = newPosition.z;
        }
    }
}
```

## Common Patterns

1. **Gizmo Scopes:** The Gizmo system heavily utilizes `IDisposable` scopes to manage rendering state. You push a transform matrix via `using ( Gizmo.Scope( myTransform ) ) { ... }`, draw your shapes, and the scope automatically pops the matrix when it closes.
2. **Immediate Mode UI:** The `Gizmo.Control` system operates as an immediate-mode GUI. You call a function like `Slider()` every frame, and it returns `true` if the user dragged it during that specific frame.
3. **Hammer Metadata:** Attributes in the `Hammer/` directory subclass `MetaDataAttribute` or `Attribute` to inject editor-only layout information (like `SphereAttribute` to draw a visual sphere radius in Hammer) into the compiled assembly metadata.

## Performance

- **Vertex Batching:** Gizmo rendering is batched. However, generating thousands of debug lines per frame via C# `OnDrawGizmos` will still incur significant managed-to-unmanaged interop overhead. Keep gizmo drawing localized to the selected object when possible.

## Troubleshooting

:::warning 
- **Gizmos Not Drawing:** Ensure that your `Component` is actively selected in the Editor (or has the `[EditorHandle]` attribute set to always draw), and that the specific Gizmo layer is enabled in the viewport toolbar.
- **Gizmo Handles Blocked:** If you draw a solid Gizmo shape (like `SolidSphere`) over a control handle, it may intercept the mouse raycast, making the handle unclickable. Draw transparent or wireframe shapes around interactive controls.
:::

## Related Pages
- [Editor & Authoring](../../editor/index.md)
- [Sandbox.Engine Index](index.md)
