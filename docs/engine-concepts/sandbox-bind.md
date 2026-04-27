---
title: Sandbox.Bind
icon: "🔌"
sources:
  - engine/Sandbox.Bind
updated: 2026-04-25
created: 2026-04-27
---

# Sandbox.Bind

> **See also:** [Sandbox.Bind (Internals)](../engine-internals/sandbox-bind.md) — engine-developer deep dive on the binding implementation.

The `Sandbox.Bind` namespace contains the data-binding system. It allows developers to link properties together so that when one property updates, the other automatically updates to match.

## Quick Start Example

This system is most commonly used within the UI (Razor) to bind data models to visual elements, but the underlying `BindSystem` can bind arbitrary objects.

```csharp
using Sandbox;
using Sandbox.Bind;

public sealed class BindExample : Component
{
    [Property] public string PlayerName { get; set; } = "Player 1";
    [Property] public string DisplayName { get; set; }

    protected override void OnStart()
    {
        // Programmatically bind DisplayName to PlayerName
        // Whenever PlayerName changes, DisplayName will automatically update
        Game.Bind.Build
            .From( this, nameof(PlayerName) )
            .To( this, nameof(DisplayName) );
    }
}
```

## Common Patterns

1. **UI Razor Binding:** While the `BindSystem` works on any object, it is heavily integrated into s&box's Razor UI system. When you write `@bind-Value={MyProperty}` in a `.razor` file, it is utilizing this underlying system to keep the UI in sync with your C# component.
2. **Polling Updates:** The system operates on a polling mechanism within its `Tick()` loop, rather than relying strictly on C# events or `INotifyPropertyChanged`. It checks the source value against a cached state.
3. **Throttling:** For performance, the engine can throttle bind checks (via `ThrottleUpdates`) so it doesn't spend excessive CPU time reflecting over thousands of bound UI properties every single millisecond.

## Related Pages
- [UI Panels](../systems/ui/index.md)
