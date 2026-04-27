---
title: "Sandbox.Event"
icon: "📅"
sources:
  - engine/Sandbox.Event/
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Event

The `Sandbox.Event` namespace provides the engine's internal global event bus.

## Quick Start Example

If you are writing a custom Editor extension or an engine subsystem, you might want to react when the engine hotloads code:

```csharp
using Sandbox;

internal class MySystem
{
    [Event.Hotload]
    public void OnHotload()
    {
        Log.Info( "Code was hotloaded, clearing my custom caches..." );
    }
}
```

## Common Patterns

1. **Static Listeners:** The most common use case is applying `[Event.Hotload]` or `[Event.Tick]` to static methods to run global logic without needing to attach a `Component` to a `GameObject`.
2. **Editor Events:** The `Sandbox.Event` system powers much of the s&box Editor's reactivity. When a user changes selection, an event is broadcast, and the Inspector panel listens to that event to update its UI.

## Visualizing Events
Engine developers can monitor the frequency and duration of events firing in real-time by opening the Editor's Profiler (`F2`). High-frequency events (like custom ticks) will show up in the CPU trace, making it easy to spot poorly performing listeners.

## Troubleshooting

:::warning
- **Event Not Firing:** Ensure the class containing the method is instantiated if it's an instance method, or that the method is marked `static`. The engine will only invoke instance methods on objects it currently tracks (like active `GameObject`s).
- **Stale Pointers on Hotload:** If you register an event listener manually and fail to unregister it before a hotload, you may create a memory leak or cause `NullReferenceException`s when the old assembly is invoked.
:::

## Sample Validation
Look at `engine/Sandbox.Test/` where various mock systems broadcast and listen to custom string events to verify the event bus correctly passes arguments and respects Priority ordering.

## Related Pages
- [Global Event System](../systems/events.md)
