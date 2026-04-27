---
title: "Sandbox.Bind"
icon: "🔗"
sources:
  - engine/Sandbox.Bind/
updated: 2026-04-25
created: 2026-04-27
---

# Sandbox.Bind

The `Sandbox.Bind` namespace contains the data-binding system, primarily utilized by the Razor UI system to observe property changes and reactively update the UI without manual polling.

## Quick Start Example

While game developers use `[Bind]` attributes implicitly within Razor, engine developers may need to interact with the underlying proxy system to trace binding resolution:

```csharp
using Sandbox.Bind;

// Build a one-way bind between two properties using BindSystem.Build.
// See engine/Sandbox.Bind/Builder.cs and BindSystem.cs.
internal class BindDebugger
{
    public void DebugBind( BindSystem system, object source, string sourceName, object target, string targetName )
    {
        system.Build
            .Set( target, targetName, onChanged: () =>
            {
                Log.Info( $"Property {targetName} changed on {target}" );
            } )
            .From( source, sourceName );
    }
}
```

## Common Patterns

1. **Proxy Instantiation:** A `BindProxy` is created for objects that need to be observed. It holds weak references to prevent memory leaks if the observer is destroyed.
2. **Link Resolution:** `Links` are the actual connections. They evaluate property paths (e.g., `Player.Inventory.ActiveWeapon.Ammo`) and manage the chain of observation so that if `Inventory` changes, the `Ammo` link is re-evaluated.

## Visualizing Binding

Engine contributors testing data bindings can view active `BindProxy` links through the Editor Inspector's diagnostic views. If a Razor panel seems to be updating continuously without user input, examining these proxy traces can isolate the `ILHotload` modification or property setter responsible for the rapid triggers.

## Troubleshooting

:::warning
- **UI Not Updating:** If the proxy system fails to detect a setter invocation (perhaps because a field was modified via reflection or native interop instead of its property setter), the UI will desync from the game state. 
- **"Bind target is null":** Happens when the root object of a binding chain is destroyed, but the link attempts to resolve. The proxy should fail gracefully rather than throwing an engine-level exception.
:::

## Sample Testing
You can review the test suite in `engine/Sandbox.Test/` specifically looking for UI binding tests to see how the proxy triggers under different mutation scenarios, verifying the exact sequence of events the hotloader injects.

## Related Pages
- [Razor UI Compiler Architecture](../systems/ui/razor/architecture.md)
