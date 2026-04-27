---
title: Sandbox.Engine.Input
icon: "🎮"
sources:
  - engine/Sandbox.Engine/Systems/Input/InputRouter.cs
  - engine/Sandbox.Engine/Systems/Input/Input.Actions.cs
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Engine.Input

The `Sandbox.Engine.Input` namespace handles the low-level processing of input devices (keyboard, mouse, gamepads, VR controllers) and translates them into the higher-level action system used by games.

## Quick Start Example

While game developers use the public `Input` API to check actions like "Jump", engine developers can query the static `InputRouter` to monitor the absolute cursor state or track active contexts.

```csharp
using Sandbox.Engine;

internal class InternalInputMonitor
{
    public void PrintCursorState()
    {
        // Conceptual: Engine code monitoring cursor state
        if ( InputRouter.MouseCursorVisible )
        {
            Log.Info( $"Cursor is visible at {InputRouter.MouseCursorPosition}" );
            Log.Info( $"Movement since last frame: {InputRouter.MouseCursorDelta}" );
        }
    }
}
```

## Input Routing

A critical responsibility of this system is determining *where* an input goes, primarily handled by the `InputRouter` located at `engine/Sandbox.Engine/Systems/Input/InputRouter.cs`. 

This is the very first place input is routed to from the native engine. From here, it attempts to route the input through an ordered chain of consumers. For example, if a developer presses a key:
1. Is an Editor panel intercepting it?
2. Does the Main Menu UI need it?
3. Is an in-game Razor UI panel focused and expecting text input?
4. If unhandled, it passes the input down to the active game Scene as a bound `InputAction`.

Engine developers working in this area must carefully manage these priorities to prevent input bleeding (e.g., jumping in the game while typing in an editor text box).

## Usage by Game Developers

Game developers rarely interact with the raw `Sandbox.Engine.Input` classes directly. Instead, they use the `Sandbox.Input` static class provided in the public API (e.g., `Input.Pressed("Jump")`), which internally queries the state managed by this engine subsystem.

If a game does not define its own custom input actions, the engine provides a set of fallbacks via `Input.Common.cs`, ensuring basic actions like "Forward", "Jump", and "Use" always exist with default keyboard and gamepad bindings.

## Related Pages
- [Detect Player Input](../../how-to/detect-input.md)
