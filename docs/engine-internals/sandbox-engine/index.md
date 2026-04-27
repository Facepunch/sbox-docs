---
title: "Sandbox.Engine"
icon: "⚙️"
sources:
  - engine/Sandbox.Engine/
updated: 2026-04-25
created: 2026-04-27
---

# Sandbox.Engine

`Sandbox.Engine` is the largest and most central managed library in the s&box architecture. It acts as the primary bridge between the raw native C++ Source 2 engine and the high-level C# game API used by developers.

## Quick Start Example

The engine's per-frame entry point lives in `Sandbox.Engine/Core/EngineLoop.cs`
(`internal static class EngineLoop`). Game-side code does not intercept this
loop directly — `RunFrame` is invoked from native and drives `FrameStart`,
`SourceEngineFrame`, and `FrameEnd` itself. To run logic each tick from game
code, hook a `Component` callback or implement a `GameObjectSystem`. See
`engine/Sandbox.Engine/Core/EngineLoop.cs` for the actual sequence.

## Common Patterns

1. **NativeEngine Wrappers:** `Sandbox.Engine` contains the hand-written C# wrappers that map to the auto-generated `Sandbox.Bind` methods. For example, a C# `Model` object holds a pointer to a native `CModel` and exposes managed properties.
2. **Context Passing:** The engine uses a GlobalContext to manage state, ensuring that the editor, the client, and the server (which may all be running in the same process during Play-In-Editor mode) do not accidentally mutate each other's memory.
3. **Yoga Layout Integration:** The UI system in `Sandbox.Engine` leverages the Yoga flexbox layout engine. When Razor parses HTML, it passes style attributes directly to Yoga nodes in native memory to calculate screen bounds efficiently.

## Visualizing Engine States

When testing changes to `Sandbox.Engine`, developers rely on the Editor's internal profiling overlays. Pressing `F2` or navigating to View -> Profiler will open detailed telemetry views generated directly from `Sandbox.Engine/Core/Diagnostics/Profiler/`. This shows the exact millisecond duration of the Scene Tick, Physics Step, and UI Layout passes.

## Troubleshooting

:::warning
- **Engine Build Failures:** Modifying `Sandbox.Engine` often requires regenerating bindings if native structs have changed. If you get `DllNotFoundException` or memory access violations, ensure your C++ Source 2 build is perfectly in sync with your C# engine branch.
- **"Game Loop Deadlock":** If an infinite loop or a synchronous blocking `Task.Wait()` is introduced in `Sandbox.Engine/Core/`, the entire editor will freeze, requiring a process kill.
:::

## Sample Validation

The majority of unit tests in `engine/Sandbox.Test.Unit/` directly target `Sandbox.Engine`. These tests validate everything from basic `Vector3` math in the Utility namespace, to complex Prefab deserialization and UI layout accuracy. Always run the test suite before submitting changes to this assembly.

## Related Pages
- [Scene System Architecture](../../systems/scene-system/architecture.md)
- [Game Architecture](../runtime-layout/game-core.md)
- [Systems (Render, Audio, Physics)](systems/index.md)
- [Input System](input.md)
