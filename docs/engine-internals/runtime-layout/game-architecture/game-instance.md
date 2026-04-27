---
title: Game Instance Lifecycle
icon: "🎮"
sources:
  - engine/Sandbox.GameInstance
updated: 2026-04-25
created: 2026-04-27
---

# Game Instance Lifecycle

The `GameInstance` manages the lifecycle of a specific game session, handling transitions, networking setup, and platform integrations. It is the boundary for the active game session.

## Quick Start Example

As a game developer, you can query the active `Game` instance to determine if the current code is executing as a server host or a client.

```csharp
using Sandbox;

public sealed class SessionChecker : Component
{
    protected override void OnStart()
    {
        if ( Game.IsServer )
        {
            Log.Info( "This instance is hosting the game." );
        }
        else if ( Game.IsClient )
        {
            Log.Info( "This instance is a connected client." );
        }
    }
}
```

## Common Patterns

1. **Client Registration:** When a new network connection is established, the `GameInstance` registers a new client to represent that player in the logic tier.
2. **String Table Management:** It manages Network String Tables, which are replicated dictionaries that map strings to integer IDs to save bandwidth over the network.

## Troubleshooting

:::warning
- **"Network String Table limit exceeded":** This means the game logic is creating too many unique networked strings. Verify that procedural generation isn't spamming the table.
:::

## Related Pages
- [Engine Internals: Sandbox.GameInstance](../../sandbox-gameinstance.md)
