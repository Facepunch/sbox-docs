---
title: "Sandbox.GameInstance"
icon: "🎮"
sources:
  - engine/Sandbox.GameInstance/
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.GameInstance

The `Sandbox.GameInstance` namespace manages the lifecycle of a loaded game, including setting up standard environments, tracking active players, and managing network string tables.

## Quick Start Example

As an engine contributor working on standalone builds or lobby systems, you might interact with the instance like so:

```csharp
using Sandbox;

internal class GameInstanceDebugger
{
    public void DebugInstance()
    {
        // Conceptual: Querying the current game instance
        if ( Game.ActiveScene != null )
        {
            Log.Info( $"Running game with {Game.Clients.Count} active clients." );
        }
    }
}
```

## Common Patterns

1. **Client Registration:** When a new network connection is established via `Sandbox.Server`, the `GameInstance` registers a new `IClient` object to represent that player in the logic tier.
2. **String Table Delta Updates:** When a game developer spawns a new Prefab, its path is added to the Network String Table. The `GameInstance` ensures this new string is broadcast to all connected clients immediately.

## Visual Diagnostics
Engine developers can monitor the `GameInstance` state via the Editor's Console by typing `status`. This dumps current memory usage, string table capacity, and active client count managed by this namespace.

## Troubleshooting

:::warning
- **"Network String Table limit exceeded":** This means the game logic is creating too many unique networked strings. Verify that procedural generation isn't spamming the table.
- **Rich Presence Desync:** If Steam friends shows a player in the Main Menu while they are in-game, verify the `RichPresence` module received the `MapLoaded` event correctly.
:::

## Sample Validation
Tests in `engine/Sandbox.Test/` related to Lobby creation, map transitioning, and string table validation cover the primary failure points of this assembly.

## Related Pages
- [Game Instance Lifecycle](../runtime-layout/game-architecture/game-instance.md)
