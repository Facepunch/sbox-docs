---
title: "ControlWidget"
icon: "⚙️"
sources:
  - engine/Sandbox.Tools/ControlWidget/ControlWidget.cs
  - engine/Sandbox.Tools/ControlWidget/FloatControlWidget.cs
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Tools.ControlWidget

The `ControlWidget` subsystem defines the base abstract UI class and implementations for every property field that appears in the Editor's Inspector pane.

## Quick Start Example

Here is a conceptual look at how a custom float slider is implemented. Notice how it extends `ControlWidget` and intercepts mouse dragging to update the underlying `SerializedProperty`.

```csharp
using Editor;

// Conceptual: A slider widget for a float property
[CustomEditor( typeof( float ) )]
internal class FloatControlWidget : ControlWidget
{
    public FloatControlWidget( SerializedProperty property ) : base( property ) { }

    protected override void OnMouseMove( MouseEvent e )
    {
        base.OnMouseMove( e );

        if ( e.ButtonState.Contains( MouseButtons.Left ) )
        {
            // Calculate pixel delta
            float delta = Application.CursorDelta.x * 0.01f;
            
            // Push the change to the serialized object
            SerializedProperty.As.Float += delta;
            
            e.Accepted = true;
        }
    }
}
```

## Advanced Patterns

### Multi-Editing

A common edge case in editor development is when the user selects *multiple* objects simultaneously (e.g., selecting 10 Point Lights and changing their color). In this scenario, the `SerializedProperty` passed to `ControlWidget.Create()` represents a `MultiSerializedObject`.

By default, widgets do not support this because writing a single value to multiple disparate memory addresses requires special handling. 

If a widget overrides `public virtual bool SupportsMultiEdit => true;`, it promises to handle `SerializedProperty.IsMultipleValues` gracefully. If it returns `false`, the factory method returns a fallback `MultiEditNotSupported` label, preventing data corruption.

## Troubleshooting

:::warning
- **Multi-Edit Data Corruption:** If you implement a custom `ControlWidget` and flag `SupportsMultiEdit => true`, ensure your `OnValueChanged` logic correctly handles `IsMultipleDifferentValues == true`. Do not blindly overwrite distinct values just because the UI was refreshed.
- **Lost UI State on Hotload:** Because `ControlWidget` instances are frequently destroyed and recreated during C# hotloading, do not store long-term state (like expanded/collapsed toggle booleans) inside the widget itself unless you serialize it into the Editor's local layout cache.
:::

## Related Pages
- [Inspector Architecture](inspector.md)
