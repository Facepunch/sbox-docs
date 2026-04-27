---
title: "Sandbox.Server.Network"
icon: "🔌"
sources:
  - engine/Sandbox.Server/Network/
updated: 2026-04-25
created: 2026-04-27
---

# Sandbox.Server.Network

The `Sandbox.Server.Network` namespace is responsible for the internal mechanics of a dedicated server's networking. While `Sandbox.Server` handles high-level session state, this specific subspace handles low-level tasks such as file sending, input processing, and the actual serialization of delta snapshots.

This page is intended for engine developers modifying how the server communicates. If you are a game developer looking to use networking in your game, see the [Server Architecture](../systems/networking-multiplayer/server-architecture.md) guide instead.

## Quick Start Example

As an engine contributor modifying how files are sent to clients during connection:

```csharp
using Sandbox.Server.Network;

internal class AddonTransferManager
{
    public void QueueFileForClient( ServerConnection conn, string filePath )
    {
        // Conceptual: Engine level hook to push a required asset to a connecting client
        var fileSend = new FileSend( conn, filePath );
        fileSend.Start();
        
        Log.Info( $"Started transferring {filePath} to client {conn.SteamId}." );
    }
}
```

## Common Patterns

1. **Delta Compression:** To avoid flooding bandwidth, the serializer maintains a history buffer for each client. When building a snapshot, it only encodes the difference between the current tick and the last tick the specific client acknowledged receiving.
2. **Throttling & Prioritization:** Not all data is created equal. Input packets are high priority, while background file transfers are throttled so they do not introduce latency into the gameplay simulation.

## Visual Diagnostics

The Editor's **Network Profiler** hooks directly into this namespace to visualize outbound bandwidth. It allows engine contributors to see exactly how many bytes `FileSend` is utilizing versus game state snapshots, aiding in identifying network bottlenecks.

## Related Pages
- [Sandbox.Server](sandbox-server.md)
- [Server Architecture](../systems/networking-multiplayer/server-architecture.md)
- [Sandbox.Client Network](sandbox-client-network.md)
