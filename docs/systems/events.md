---
title: Global Event System
icon: "⚡"
sources:
  - engine/Sandbox.Event/EventSystem.cs
created: 2026-04-27
updated: 2026-04-27
---

# Global Event System

A low-level, reflection-based global event bus used primarily by the engine internally for lifecycle broadcasts.

## Quick Start Example

```csharp
using Sandbox;

public sealed class GlobalListener : Component
{
    // The method must be public, static, and tagged with [Event]
    [Event("client.connected")]
    public static void OnClientConnected()
    {
        Log.Info( "A client connected to the server!" );
    }
}
```

> **Prefer `ISceneEvent<T>` for game code.** Scene Events are strongly typed, faster, and work within the `GameObject` hierarchy. The Global Event System is mainly for engine-internal lifecycle hooks where no Scene context exists.

1. **Listening to Engine Hooks:** The primary use case for game developers is listening to built-in engine events that happen outside the scope of a specific Scene. For instance, reacting to a Hotload compilation event or a client joining the lobby.
2. **Static Scope:** Methods marked with `[Event]` must be `static`. They do not belong to an instance of an object, meaning you cannot access instance variables (like `this.Transform`) directly from within the event callback.

## Troubleshooting

:::warning 
- **Method Not Firing:** Ensure your method is `public` and `static`. If it is an instance method, the event system will fail to invoke it via reflection and silently skip it.
- **Typo in Event Name:** Because the routing is string-based, a typo in `[Event("clent.connected")]` will cause the method to never fire. Always double-check the event string against the official documentation or source code.
:::

## Related Pages
- [Scene Events](../scene/components/events/index.md)
