---
title: "Editor Input Bindings"
icon: "⚙️"
sources:
  - engine/Sandbox.Tools/Input/InputSystem.cs
updated: 2026-04-25
created: 2026-04-27
---

# Editor Input Bindings

This is a single file — `engine/Sandbox.Tools/Input/InputSystem.cs` — and a single public method. Its job is to let editor code read the current input-action list without taking a hard dependency on `Sandbox.Engine.Input`.

```csharp
using Sandbox;

namespace Editor;

public static partial class InputSystem
{
    public static IReadOnlyList<InputAction> GetCommonInputs() => Sandbox.Engine.Input.CommonInputs;
}
```

That's the whole file. It's `partial` so other tools-side files can extend `Editor.InputSystem` with their own helpers without modifying engine code.

## When you'd touch this

You'd add to this file when an editor tool needs to display or interact with the configured input actions — typically a tooltip ("Press Space to Jump"), a keybind UI, or a viewport overlay that shows the current control scheme. The expected pattern is to add a new helper method here that delegates to `Sandbox.Engine.Input` rather than reaching across the boundary directly.

## Related

- [Input System (Game-dev side)](../../systems/input/index.md) — what `InputAction` and `Sandbox.Engine.Input` actually are.
