---
title: "Sandbox.Engine: Networking"
icon: "📡"
sources:
  - engine/Sandbox.Engine/Systems/Networking/Networking.cs
  - engine/Sandbox.Engine/Systems/Networking/Networking.Thread.cs
  - engine/Sandbox.Engine/Systems/Networking/Networking.LobbyQuery.cs
  - engine/Sandbox.Engine/Systems/Networking/System/Channel/Connection.Input.cs
created: 2026-04-27
updated: 2026-04-27
---

# Networking

The `Networking` subsystem within `Sandbox.Engine` provides the low-level foundation for connecting clients together, running servers, and routing multiplayer traffic. It manages `SteamNetwork` connections, connection states, fake lag testing, and Steam lobbies.

## Architecture

This subsystem acts as the engine's interface to underlying transports (primarily Steam Networking Sockets via `SteamNetwork` or local `TcpSocket`).

The global static `Networking` class is the primary entry point for managing connections to remote servers or creating lobbies.

## Message Recording & ZSTD Dictionary Training

The internal network system provides a mechanism to record raw network messages via reservoir sampling for `zstd` dictionary training.
This is heavily used to optimize network bandwidth consumption by creating efficient compression dictionaries tailored to a game's specific packet distribution.

*   `Networking.TryRecordMessage()`: Intercepts raw message bytes on the hot path. If recording is active, it adds them to a fixed-size `_reservoir` array. Reservoir sampling guarantees that the final set of messages accurately reflects the probability distribution of message types seen across the entire session, without unbounded memory growth.
*   `net_msgrecord [capacity]`: A protected console command that begins the sampling process.
*   `net_msgrecord_stop`: Halts recording and flushes the `byte[]` arrays to disk under `netmsg_samples/`.

These binary dumps can then be fed into the external `zstd --train` utility to generate an optimal dictionary.

## UserCommands

Input events are serialized across the network as `UserCommand` structs.
*   The `UserCommand` packs a `CommandNumber` and a `ulong` bitmask of `Actions` into a tight `ByteStream`.
*   These are batched and dispatched over the wire natively to ensure client input is received cleanly on the server tick.

Here is how `Sandbox.Input.Actions` gets packed and sent across the wire:

```csharp
using Sandbox;

public abstract partial class Connection
{
    // 1. Pack the global Input Actions into the struct
    internal void BuildUserCommand( ref UserCommand cmd )
    {
        cmd.Actions = Sandbox.Input.Actions;
    }
    
    // 2. Unpack the UserCommand on the server to update the Connection's input state
    internal struct InputState
    {
        public void ApplyUserCommand( in UserCommand cmd )
        {
            var commandNumberDelta = cmd.CommandNumber - _lastUserCommand.CommandNumber;

            // Drop duplicates or commands that are too far behind
            if ( commandNumberDelta is 0 or > 0x7FFFFFFF )
                return;
                
            // ... determine pressed / released bitmasks
            Actions = cmd.Actions;
            _lastUserCommand = cmd;
        }
    }
}
```

## Threading & Ticks

To prevent heavy render frames from causing network drops or missed packets, the actual socket interaction operates asynchronously.
*   **`Networking.Thread.cs`:** Manages a dedicated high-priority background thread.
*   **The Loop:** It continuously invokes `System.ProcessMessagesInThread()` using a `ManualResetEventSlim` to sleep exactly enough time to maintain the target network tick rate (`s_threadTickRate`, default 30).
*   **Main Thread Sync:** `PreFrameTick()` and `PostFrameTick()` are called on the main application thread to safely pass buffered structural updates (like Component tables) down to the background thread's execution queues.

## Example: Joining a Lobby

The networking layer can query Steam lobbies and connect to them automatically.

```csharp
using Sandbox.Engine;

// Queries Steam for a lobby matching the ident, then connects
internal async Task AutoJoin( string gameIdent )
{
    Log.Info( $"Searching for lobbies running {gameIdent}..." );
    
    bool success = await Networking.JoinBestLobby( gameIdent );
    
    if ( success )
    {
        Log.Info( "Connected!" );
    }
    else
    {
        Log.Warning( "Failed to find or connect to a lobby." );
    }
}
```

## Example: Creating a Lobby

You can host a game session using `Networking.CreateLobby()`.

```csharp
using Sandbox.Engine;

internal void HostGame()
{
    var config = new LobbyConfig
    {
        Name = "My Custom Server",
        MaxPlayers = 16,
        Privacy = LobbyPrivacy.Public
    };

    // Starts the lobby host.
    // If we're a dedicated server, this uses DedicatedServer.Start()
    Networking.CreateLobby( config );
}
```

## Fake Lag

For testing network conditions in the editor, the system supports simulating bad connections. `Networking.UpdateFakeLag()` monitors developer console variables (`net_fakelag_ms`, `net_drop`, etc.) and configures the `SteamNetwork` interface to simulate latency and packet drops.

## Related Components
- [Scene Network System](../scene-networking.md) - The higher-level system that synchronizes GameObjects and Components over these sockets.
