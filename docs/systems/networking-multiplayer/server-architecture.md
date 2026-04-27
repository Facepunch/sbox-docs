---
title: Server Architecture
icon: "🌐"
sources:
  - engine/Sandbox.Server/
  - engine/Sandbox.Engine/Systems/Networking/
created: 2026-04-27
updated: 2026-04-27
---

# Server Architecture

The Server Architecture acts as the authoritative center of a multiplayer game in s&box, managing incoming connections, serving compiled assemblies to clients, and dictating the authoritative game state through replication and delta snapshots.

## Quick Start Example

As a game developer, you can query whether your code is running on the authoritative host (either a Dedicated Server, or the host player of a Listen Server) to perform privileged actions like spawning AI or validating scores:

```csharp
using Sandbox;

public sealed class ServerCheck : Component
{
    protected override void OnStart()
    {
        // Networking.IsHost is true if we are the authoritative server
        if ( Networking.IsHost )
        {
            Log.Info( "We are the server! Spawning authoritative enemies..." );
            // E.g., Instantiate a Prefab and call GameObject.NetworkSpawn()
        }
    }
}
```

## Common Patterns

1. **Network Spawning:** GameObjects created purely via `new GameObject()` on the server are *local* to the server. The server must explicitly call `.NetworkSpawn()` to tell the architecture to assign it a Network ID and begin replicating it to clients.
2. **Authoritative Logic:** The server never trusts the client. Movement inputs are sent from the client as commands, but the server applies them to the authoritative physics body. If the client diverges, the server's state overwrites the client's.
3. **RPC Ownership:** The `[Rpc.Owner]` attribute allows the server to send a targeted method call only to the specific client that "owns" a GameObject (like their player pawn), saving bandwidth.

## Troubleshooting

:::warning
- **"NetworkSpawn failed: Object not registered":** You must call `.NetworkSpawn()` after the object and its components are fully configured, and the object must be active in the Scene.
- **Client Desync (Rubberbanding):** If your server logic applies physics forces that the client's prediction logic cannot replicate (e.g., relying on a server-only random number generator), the client will constantly desync and snap back to the server's position. Always use networked or seeded RNG.
:::

## Related Pages
- [Client Architecture](client-architecture.md)
- [Sandbox.Server](../../engine-internals/sandbox-server.md)
