---
title: "Sandbox.Client"
icon: "💻"
sources:
  - engine/Sandbox.Client/
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Client

The `Sandbox.Client` namespace orchestrates the local engine's connection state, local predictions, and client-specific networking protocols that bridge C# with the underlying Source 2 network layer.

## Quick Start Example

As an engine contributor modifying how the client connects to servers, you might work with the internal network interceptors:

```csharp
using Sandbox.Client.Network;

internal class ClientConnectionDebugger
{
    public void HookConnection( ClientConnection conn )
    {
        // Conceptual: Engine level hook into the raw client connection
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

1. **Delta Compression Reading:** The server sends only what has changed (delta compression). `Sandbox.Client` unpacks this binary data and updates the managed C# `GameObject` properties accordingly.
2. **RPC Dispatch:** When a Remote Procedure Call arrives from the server, this layer decodes the method signature, finds the target `Component` via its network ID, and invokes the method.

## Visual Diagnostics
If you need to analyze snapshot sizes or prediction errors visually, the s&box Editor's "Network Inspector" tab exposes data driven by `Sandbox.Client.Network`. You can see byte sizes per property over time. Engine contributors use this tool to optimize delta compression sizes when making underlying networking changes.

## Troubleshooting

:::warning
- **"Snapshot too large":** If the server sends an update that exceeds the maximum UDP packet size and the client's assembler fails to stitch it together, the client will disconnect. Engine developers must ensure delta compression handles massive scene changes gracefully.
- **Prediction Errors (Rubberbanding):** If the client's local logic modifies a property that is *not* marked as predictable, the server will constantly overwrite it, and `Sandbox.Client` will snap the value back, causing visible jitter.
:::

## Sample Validation
Review tests within `engine/Sandbox.Test/` that simulate lag, packet loss, and high entity counts to validate changes to `Sandbox.Client.Network` snapshot unpacking logic.

## Related Pages
- [Client Architecture](../systems/networking-multiplayer/client-architecture.md)
- [Sandbox.Server](sandbox-server.md)
