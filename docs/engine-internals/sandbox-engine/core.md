---
title: Sandbox.Engine.Core
icon: "🧠"
sources:
  - engine/Sandbox.Engine/Core/Bootstrap.cs
  - engine/Sandbox.Engine/Core/EngineLoop.cs
  - engine/Sandbox.Engine/Core/ErrorReporting/ErrorReporter.cs
updated: 2026-04-25
created: 2026-04-27
---

# Sandbox.Engine.Core

The `engine/Sandbox.Engine/Core/` directory contains the foundational systems that initialize the s&box engine, manage its main loop, handle performance profiling, and report critical errors. It acts as the very first layer of managed C# code that the underlying native C++ engine (Source 2) interacts with.

## Quick Working Example

While standard game developers won't interact with the Core namespace directly, engine contributors often modify `EngineLoop.cs` to inject logic directly into the main frame tick:

```csharp
using Sandbox;
using Sandbox.Utility;

// Pattern used inside engine/Sandbox.Engine/Core/EngineLoop.cs:
// hold a reusable Superluminal instance and call .Start() in a using block.
internal static class CustomEngineLoop
{
    static Superluminal _customTick = new Superluminal( "CustomEngineTick", "#4d5e73" );

    internal static void TickFrame()
    {
        using ( _customTick.Start() )
        {
            // Read the time delta since the last frame
            float delta = Time.Delta;

            // Do high-priority engine processing...
        }
    }
}
```

## Common Patterns

1. **Superluminal Scopes:** You will see `using var _ = new Superluminal("EventName")` extensively throughout the Core codebase. This is a highly optimized, zero-allocation profiler marker that reports directly back to the native profiling tools.
2. **SkipHotload Attribute:** Almost all classes in the `Core/` directory are decorated with `[SkipHotload]`. Because these classes manage the lifecycle of the engine itself, replacing them in memory while the engine is running would cause a fatal crash.

## Performance

- **Zero Allocation Required:** The `EngineLoop.RunFrame` is the hottest path in the entire C# codebase. Any memory allocations (e.g., creating a new object or string) within this loop will immediately trigger the Garbage Collector, causing the engine to stutter. Engine developers must strictly adhere to using value types, structs, and pre-allocated buffers in this namespace.
- **Synchronous Execution:** The `EngineLoop` is strictly single-threaded to synchronize with the underlying Source 2 native render and physics pipelines.

## Troubleshooting

:::warning 
- **"Engine Crashed without Stack Trace":** If you make a mistake in `Bootstrap.cs` before the `ErrorReporter` system is fully initialized, the engine will crash silently. You must use native debuggers (like Visual Studio attached to the C++ process) to diagnose these early-boot failures.
- **Deadlocks in EngineLoop:** Using blocking calls like `Task.Wait()` or `Thread.Sleep` inside the `EngineLoop` will instantly freeze the entire Editor and Game window.
:::

## Related Pages
- [Engine Launcher](../launcher.md)
- [Sandbox.Engine Index](index.md)
