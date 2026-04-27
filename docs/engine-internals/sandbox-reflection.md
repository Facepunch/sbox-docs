---
title: "Sandbox.Reflection"
icon: "🔍"
sources:
  - engine/Sandbox.Reflection/
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Reflection

The `Sandbox.Reflection` namespace provides the engine's custom `TypeLibrary`, which is a highly optimized wrapper around standard .NET reflection used to discover types and properties at runtime.

## Quick Working Example

As an engine developer, you will frequently use `TypeLibrary` to interact with user-defined components without knowing their types at compile time (e.g., when building the Editor Inspector).

```csharp
using Sandbox;

internal class InspectorBuilder
{
    public void BuildUIForComponent( Component comp )
    {
        // 1. Get the cached TypeDescription
        var typeDesc = TypeLibrary.GetType( comp.GetType() );
        
        // 2. Iterate through properties marked with [Property]
        foreach ( var prop in typeDesc.Properties.Where( x => x.HasAttribute<PropertyAttribute>() ) )
        {
            Log.Info( $"Found editable property: {prop.Name} of type {prop.PropertyType}" );
            
            // 3. Read the value quickly without standard reflection overhead
            var currentValue = prop.GetValue( comp );
        }
    }
}
```

## Troubleshooting

:::warning
- **"Type not found in TypeLibrary":** If a class is dynamically generated at runtime (via `Reflection.Emit`) or comes from a banned assembly that the analyzer skipped, the `TypeLibrary` won't know about it.
- **Stale Type References:** If engine code caches a raw `System.Type` directly instead of looking it up via `TypeLibrary` by name, that cached type will become invalid (and point to a dead assembly) after a hotload.
:::

## Related Pages
- [Type Library & Reflection](../scripting-code-workflows/reflection.md)
