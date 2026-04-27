---
title: "Sandbox.Engine: Hotload"
icon: "🔄"
sources:
  - engine/Sandbox.Engine/Systems/Hotload/HotloadManager.cs
updated: 2026-04-25
created: 2026-04-27
---

# Hotload

The `Hotload` subsystem within `Sandbox.Engine` powers the fast iteration cycle of s&box, allowing C# code to be compiled, loaded, and applied to running instances without restarting the engine or losing game state.

## Example: Triggering a Swap Programmatically

Typically, the hotload runs automatically when a file is saved. However, engine or tools code may interact with the manager to trigger replacements explicitly.

```csharp
using Sandbox;
using System.Reflection;

// HotloadManager is internal to Sandbox.Engine and owned per-package by
// the PackageLoader. From engine code, you obtain an instance and call
// Replace() to queue an assembly swap, then DoSwap() to apply it.
// See engine/Sandbox.Engine/Systems/Hotload/HotloadManager.cs and
// engine/Sandbox.Engine/Services/Packages/PackageManager/PackageLoader.cs.
internal void TriggerCodeSwap( HotloadManager manager, Assembly oldAssm, Assembly newAssm )
{
    manager.Replace( oldAssm, newAssm );

    if ( manager.NeedsSwap )
    {
        // Performs the actual heap walk; may cause a frame hitch.
        manager.DoSwap();
    }
}
```

## Watched and Ignored Assemblies

The `HotloadManager` does not watch every assembly loaded into the application domain. This is for performance reasons, as walking the heap for unchanged library types would be extremely slow.

### Watched
Assemblies that contain highly volatile game code or engine abstractions that developers frequently change.
*   `Sandbox.Engine`
*   `Sandbox.System`
*   Any custom Addon or Game assemblies loaded dynamically.

### Ignored
Base libraries and third-party dependencies are explicitly ignored, as their schemas do not change during an engine session.
*   `NLog`
*   `LiteDB`
*   `SkiaSharp`
*   `System.Net.Http`

## Advanced Upgraders

When the system swaps types, it tries to map the old fields to the new fields automatically. If a type's memory layout drastically changes, or if it holds critical unmanaged state, `CodeUpgraders` are used.

Certain engine types are skipped entirely during the instance upgrading process if they are known to cause issues (e.g., `ExceptionDispatchInfo`, internal `JsonNode` types).

During `DoSwap()`, the `HotloadManager` uses the `HeavyGarbageRegion` struct and pins threads. It logs a warning via the `Log.Error` API if a single type takes too long to hotload (e.g., if there are thousands of instances of a massive struct).

*   **Instance Queue Time:** Finding all reachable object graphs.
*   **Static Field Time:** Patching static states across the AppDomain.
*   **Processing Time:** The raw time spent memcpy'ing state from the old type format to the new.
