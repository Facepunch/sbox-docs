---
title: "Sandbox.Client Network"
icon: "📶"
sources:
  - engine/Sandbox.Client/Network/
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Client Network

The `Sandbox.Client.Network` namespace contains the low-level internals for how the s&box client connects to a server, establishes a session, and synchronizes state over the network. 

This page details the underlying engine mechanisms. If you are a game developer looking to use networking in your game, refer to the [Client Architecture](../systems/networking-multiplayer/client-architecture.md) guide instead.

## Quick Start Example

As an engine contributor, you might need to hook into the raw client connection to inspect packet flow or intercept specific commands.

```csharp
using Sandbox.Client.Network;

internal class NetworkInterceptor
{
    public void AttachToConnection( ClientConnection conn )
    {
        // Conceptual: Engine level hook to inspect raw packets
        conn.OnPacketReceived += ( packet ) =>
        {
            if ( packet.Type == PacketType.StringCmd )
            {
                Log.Info( "Received string command from server." );
            }
        };
    }
}
```

## Common Patterns

1. **RPC Dispatch:** When the server broadcasts a Remote Procedure Call, the client network layer deserializes the method signature, identifies the target Component via its network ID, and invokes the function locally.
2. **Clock Syncing:** Smooth prediction requires the client's clock to perfectly match the server's tick rate. The network layer continuously calculates latency and adjusts a local time multiplier to prevent buffer overruns or underruns.

## Troubleshooting

:::warning Common Gotchas
- **"Snapshot too large":** If a server update exceeds the maximum UDP packet size and the client assembler fails, the connection drops. Engine developers must ensure delta compression handles massive scene changes gracefully.
- **Rubberbanding/Prediction Errors:** Occurs when the client modifies a property not marked as predictable, causing a constant tug-of-war with the server's authoritative state.
:::

## Visual Diagnostics

Engine contributors can analyze snapshot sizes and prediction errors using the Editor's **Network Inspector** tab. This tool, driven by `Sandbox.Client.Network`, breaks down byte usage per property over time, which is critical for optimizing delta compression algorithms.

## Related Pages
- [Sandbox.Client](sandbox-client.md)
- [Client Architecture](../systems/networking-multiplayer/client-architecture.md)
- [Sandbox.Server](sandbox-server.md)
