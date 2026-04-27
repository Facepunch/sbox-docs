---
title: Sandbox.AppSystem
icon: "🧠"
sources:
  - engine/Sandbox.AppSystem
updated: 2026-04-25
created: 2026-04-27
---

# Sandbox.AppSystem

> **See also:** [Sandbox.AppSystem (Internals)](../engine-internals/sandbox-appsystem.md) — engine-developer deep dive on lifecycle management.

The `Sandbox.AppSystem` namespace provides the core architecture for initializing and managing the lifecycle of distinct engine modules (like rendering, physics, or input) during the engine's startup sequence.

## Quick Start Example

You do not interact with the `AppSystem` directly when creating games in s&box. It is used exclusively by the engine's internal bootstrap process.

```csharp
using Sandbox;

// Conceptual example of how the engine registers an AppSystem
// This is an internal engine pattern, not used in game addons
internal class PhysicsAppSystem : Sandbox.AppSystem
{
    public override void Init()
    {
        // Initialize the native physics library
        NativeEngine.Physics.Initialize();
        Log.Info( "Physics System Initialized" );
    }

    public override void Shutdown()
    {
        // Cleanup native memory when the engine closes
        NativeEngine.Physics.Shutdown();
    }
}
```

## Common Patterns

1. **Lifecycle Hooks:** Classes inheriting from `AppSystem` implement methods like `Init()`, `PostInit()`, and `Shutdown()`.
2. **Dependency Management:** The engine uses `MissingDependancyDiagnosis.cs` to ensure that if System B requires System A to function, System A is initialized first. If a required native Source 2 module is missing from the `bin/` directory, the AppSystem catches it and throws a formatted error.
3. **Flavors of Execution:** Different executables utilize different subsets of AppSystems. For instance, `EditorAppSystem.cs` initializes Qt and the toolset UI, while `StandaloneAppSystem.cs` skips the editor tools entirely and just boots the game renderer.

- **Missing Native DLLs:** The most common issue handled by the AppSystem is a missing native `.dll`. If `engine2.dll` or `rendersystemdx11.dll` fails to load, the `AppSystem` halts execution and uses native Windows `MessageBox` calls to alert the user, as the normal C# UI system cannot function without those libraries.

## Related Pages
- [Engine Launcher](launcher.md)
