---
title: "Modes"
icon: "📋"
created: 2024-12-06
updated: 2025-08-11
---

# Modes

Modes are different ways your shader will render under different passes or conditions.

A mode can be specified in three ways:

* `Mode()`
* `Mode( S_MODE_COMBO )` - sets the combo when this mode is active
* `Mode( "mode_fallback.shader" )` - uses a different shader for this mode

Your typical modes section might look like:

```csharp
MODES
{
    Forward();
    Depth();
}
```

## Supported Modes

| `Default` | Used for compute shaders |
|---------|--------------------------|
| `Forward` | Standard rendering       |
| `Depth` | Depth prepass, shadows   |
| `ToolsShadingComplexity` | Shows Quad Overdraw      |
