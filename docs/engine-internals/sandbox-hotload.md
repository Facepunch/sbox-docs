---
title: "Sandbox.Hotload"
icon: "🔄"
sources:
  - engine/Sandbox.Hotload/
updated: 2026-04-25
created: 2026-04-27
---

# Sandbox.Hotload

The `Sandbox.Hotload` namespace is responsible for the engine's signature feature: hot-swapping compiled C# assemblies while the game is running, without losing simulation state.

## Quick Start Example

As a game developer or engine contributor, you generally write code to *react* to a hotload rather than driving it directly:

```csharp
using Sandbox;
using System.Collections.Generic;

// Implement IHotloadManaged on a type to receive hotload callbacks.
// See engine/Sandbox.System/Attributes/Hotload.cs.
public sealed class HotloadReceiver : Component, IHotloadManaged
{
    void IHotloadManaged.Destroyed( Dictionary<string, object> state )
    {
        // Optionally stash values for the replacement instance.
    }

    void IHotloadManaged.Created( IReadOnlyDictionary<string, object> state )
    {
        Log.Info( "Code was just hotloaded! Re-initializing cached native pointers." );
        // Clear/restore stale references here.
    }
}
```

## Common Patterns

1. **ILHotload:** Sometimes a user only changes the body of a method, rather than adding fields or renaming classes. The `ILHotload` subsystem attempts to use .NET `EnC` (Edit and Continue) to hot-patch the Intermediate Language directly in memory, bypassing the need to serialize/deserialize the Scene entirely. This is blisteringly fast.
2. **Property Upgraders:** When parsing serialized states, `Sandbox.Hotload/Properties/` ensures that if an `int` was changed to a `float` in code, the data isn't corrupted during re-injection.

## Visual Diagnostics

The s&box Editor provides instant feedback on the hotload process via the Console. A green message indicates a fast `ILHotload`, while a full state rebuild is often accompanied by an `[Event.Hotload]` log trace if developers have hooked it. Engine contributors use these traces to ensure complex engine subsystems (like the physics world) successfully restored their state after the memory swap.

## Troubleshooting

:::warning
- **"Assembly Load Context cannot unload":** This is a critical error for engine developers. It means a reference to an old user type is still being held somewhere in `Sandbox.Engine` or native memory. Finding the leak often requires dumping the .NET memory heap and tracing GC roots.
- **Data Loss on Rename:** If a user renames a `[Property]` without using the `[Obsolete]` tag or a proper migration path, the hotloader won't know where to put the old data, and it will reset to its default value on compile.
:::

## Sample Validation

The hotload subsystem is rigorously tested in `engine/Sandbox.Test.Hotload/`. These tests simulate modifying C# code dynamically, triggering the hotload pipeline, and asserting that values in memory survived the transition correctly. Validating against these tests is mandatory before altering `Sandbox.Hotload` logic.

## Related Pages
- [Sandbox.Compiling](sandbox-compiling.md)
- [Code Hotloading & Compilation](../scripting-code-workflows/code-hotloading.md)
