---
title: Code Hotloading & Compilation
icon: "🔁"
sources:
  - engine/Sandbox.Hotload
  - engine/Sandbox.Compiling
updated: 2026-04-25
created: 2026-04-27
---

# Code Hotloading & Compilation

> **You're on the architectural overview** — how the compile + deep-swap pipeline is structured, the `[SkipHotload]` attribute, auto-skipped types, the `[Event("hotload")]` listener. For day-to-day iteration tips, the IL fast-hotload path, diagnosis with `hotload_log 2`, and the gotchas (default values, dictionaries, delegates, generics, reflection caches) see [Hotloading (developer guide)](../code/code-basics/hotloading.md).

s&box provides near-instant development iteration by automatically compiling your C# code in memory and swapping the assemblies without tearing down the game state.

## Quick Start Example

Hotloading is automatic. However, you can listen for it using events.

```csharp
using Sandbox;

public sealed class HotloadListener : Component
{
    [Event("hotload")]
    public void OnHotload()
    {
        Log.Info( "Code was just recompiled and hotloaded!" );
    }
}
```

## Skipping Hotload

Sometimes, deep swapping an object is unnecessary, undesirable, or dangerous (for example, native pointers). You can use the `[SkipHotload]` attribute to tell the engine's Hotloader to ignore specific types or fields during the swap.

```csharp
using Sandbox;

[SkipHotload]
public class MyNativeWrapper
{
    // The Hotloader will skip this entire class, 
    // leaving instances untouched in memory.
}

public class MyComponent : Component
{
    [SkipHotload]
    public object HugeCacheData; // The Hotloader will not copy this field to the new instance
}
```

### Auto-Skipped Types

The engine's `SkipUpgrader` automatically skips Hotloading for many fundamental C# types because they do not require deep swapping and are safe to leave alone. You don't need to add `[SkipHotload]` to:
- `string`, `Guid`, `Decimal`
- Standard generic collections: `Dictionary<,>`, `List<>`, `Queue<>`, `HashSet<>`
- Thread-safe collections: `ConcurrentDictionary<,>`
- References: `WeakReference<>`, `ConditionalWeakTable<,>`

### Edge Cases

- **Generic Types:** If a generic type argument contains an unsealed reference type (that isn't primitives or value types), it might not be skipped automatically because it could contain deriving types that need hotloading.
- **Inheritance:** `[SkipHotload]` is **not inherited**. If you place it on a base class, fields added by derived classes will still be evaluated by the Hotloader unless the derived class also specifies `[SkipHotload]`.
- **Statics:** If you add `[SkipHotload]` to a static field, it will retain its value across reloads instead of being re-initialized or copied to a new backing field. This is heavily used in core engine subsystems.

## Common Patterns

1. **Seamless Iteration:** Change logic, add `[Property]` fields, or alter constants and see results immediately without losing game state.
2. **State Reset:** If you want a specific subsystem to completely reset on hotload instead of keeping its state, hook into the `[Event("hotload")]` event and clear your dictionaries/lists manually.

## Related Pages

- [Hotloading (developer guide)](../code/code-basics/hotloading.md) — pitfalls, IL fast-hotload, diagnosis with `hotload_log 2`, and the rules for default values, dictionaries, generics, delegates, and reflection caches.
