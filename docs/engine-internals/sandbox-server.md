---
title: "Sandbox.Server"
icon: "🌐"
sources:
  - engine/Sandbox.Server/
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Server

The `Sandbox.Server` namespace handles internal dedicated-server initialization, client connection management, snapshot compression, and network state authority.

## Quick Start Example

As an engine contributor working on server administration, you might hook into connection pipelines:

```csharp
using Sandbox.Server;

internal class ConnectionDebugger
{
    public void OnClientConnected( ServerConnection conn )
    {
        // Conceptual: Engine level hook into raw server connections
        Log.Info( $"New client connected with SteamID {conn.SteamId} from IP {conn.IpAddress}" );
    }
}
```

## Common Patterns

1. **Client Disconnect Handling:** When a client drops (due to a timeout, crash, or graceful exit), the `Server` subsystem ensures all GameObjects "owned" by that client are either destroyed or reassigned to the server automatically to prevent stale state.
2. **Bandwidth Throttling:** `Sandbox.Server` actively monitors the bytes sent to each client. If a client connection is struggling, it may drop snapshots or lower update rates for non-essential objects.

## Visual Diagnostics

Engine contributors can visualize the server's networking load using the Network Overlay in-game or the Editor's Network Inspector. This will display the exact byte counts processed by `Sandbox.Server` per tick, helping isolate if a specific Component is flooding the network queue.

## Troubleshooting

:::warning
- **"Client timed out":** This happens if the server's main thread blocks for too long (e.g., executing a massive `while` loop in a Component). The network thread will fail to send keep-alive packets, and clients will drop.
- **"Invalid Auth Ticket":** Ensure that the Steam API is initialized on the dedicated server. If running locally without Steam, ensure the server is configured to accept LAN connections (`-insecure`).
:::

## Sample Validation
Look at `engine/Sandbox.Test/` where mock server connections are spun up. These tests validate that the server correctly drops malicious packets, handles rapid disconnects, and generates accurate baseline snapshots without crashing.

## Related Pages
- [Server Architecture](../systems/networking-multiplayer/server-architecture.md)
- [Client Architecture](../systems/networking-multiplayer/client-architecture.md)
