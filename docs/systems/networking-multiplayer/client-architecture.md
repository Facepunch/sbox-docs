---
title: Client Architecture
icon: "📺"
sources:
  - engine/Sandbox.Client/
  - engine/Sandbox.Engine/Systems/Networking/
created: 2026-04-27
updated: 2026-04-27
---

# Client Architecture

The client architecture handles the local player's connection to dedicated servers or listen servers, managing assembly downloads, prediction, and memory-backed asset storage.

## Quick Start Example

When writing game code, you can easily verify your network state and react to client-side specific logic:

```csharp
using Sandbox;

public sealed class ClientCheck : Component
{
    protected override void OnStart()
    {
        // This will be true if you are a connected client in a multiplayer game
        if ( Networking.IsActive && !Networking.IsHost )
        {
            Log.Info( "We are connected to a remote network session as a client!" );
            // You can safely setup UI or local predictive effects here
        }
    }
}
```

## Common Patterns

1. **Client-Side Prediction:** Because network latency exists, waiting for the server to confirm movement makes the game feel sluggish. The client runs the exact same `PlayerController` code as the server. If the server later sends a correction (because the client hit an obstacle they didn't know about), the client architecture smoothly interpolates the visual error.
2. **RPCs (Remote Procedure Calls):** Clients can define methods with `[Rpc.Broadcast]` or `[Rpc.Owner]`. The networking system intercepts these calls on the client, serializes the arguments, and transmits them over the transport layer (TCP or Steam Datagram Relay).
3. **Transport Agnostic:** The engine supports multiple transports (`SteamLobby`, `SteamNetworkSockets`, `Tcp`). The `GameNetworkSystem` abstracts these away so game developers just write `Connection.Send()`.

## Troubleshooting

:::warning
- **"Server is running a different version":** If the server's addon code compilation hash does not match the client's (often due to the server not automatically updating its addons), the client architecture will gracefully disconnect them rather than risk memory corruption.
- **"Ghosting / Stuttering":** Ensure you are using `Time.Delta` correctly in `OnUpdate` and `Time.FixedDelta` in `OnFixedUpdate`. The prediction engine cannot smooth out logic that uses the wrong time step.
:::

## Related Pages
- [Sandbox.Client](../../engine-internals/sandbox-client.md)
- [Server Architecture](server-architecture.md)
