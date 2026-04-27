---
title: "Platform"
icon: "🖥️"
sources:
  - engine/Sandbox.Engine/Platform/
updated: 2026-04-25
created: 2026-04-27
---

# Platform Integration

The Platform system in s&box handles integrations with underlying host environments such as Steam, Standalone builds, VR runtimes, and live streaming services like Twitch.

## Quick Start Example

```csharp
using Sandbox;

public sealed class PlatformCheckComponent : Component
{
    protected override void OnStart()
    {
        // Check if the game is running as a standalone build
        if ( Application.IsStandalone )
        {
            Log.Info( "Running standalone build." );
        }
        else
        {
            // Local Steam display name comes through the Connection
            var playerName = Connection.Local?.DisplayName ?? "(unknown)";
            Log.Info( $"Running on Steam as {playerName}" );
        }
    }
}
```

## Core Platform Subsystems

### Standalone
When you export your game to distribute outside of the s&box ecosystem, it runs as a **Standalone** build. The `Standalone` API provides access to the build manifest, build dates, and handles initializing the application environment without the s&box developer editor. 
* *Use Case:* Checking if your game is running in a published `.exe` versus the editor to toggle debug tools.

### Steam
The engine natively integrates with Steam through the Steamworks API (`engine/Sandbox.Engine/Platform/Steam/`). This provides access to:
*   Friends lists and rich presence.
*   Matchmaking and lobbies.
*   Steam Datagram Relay (SDR) for networked multiplayer.
*   Steam Input for controller support.

### Stream (Twitch Integration)
s&box includes native support for streaming platforms, primarily Twitch (`engine/Sandbox.Engine/Platform/Stream/`). You can hook into chat messages via IRC, stream predictions, and channel rewards directly from your game code.
* *Use Case:* Spawning an enemy in your game whenever a viewer types a specific command in Twitch chat, or creating interactive polls.

### VR (Virtual Reality)
The VR platform subsystem (`engine/Sandbox.Engine/Platform/VR/`) connects the engine to OpenXR. It provides the low-level inputs for headset tracking, controller poses, and overlay rendering. Note that you typically won't interact with this raw platform API directly; instead, you'll use the higher-level [VR Components](../vr/index.md) (like `VRAnchor` or `VRHand`) provided in the Scene system.

## Common Patterns

1. **Abstracting Platform Logic:** If your game relies heavily on Steam-specific features (like lobbies), ensure you have fallback logic for when `Application.IsStandalone` is true but the user isn't connected to Steam.
2. **Streaming Interactivity:** When using the `Stream` API, always validate that the streamer is actually live and connected before relying on chat events to drive game logic.

## Troubleshooting

:::warning
- **"Steam isn't initializing in my Standalone build!"**
  If you are running a standalone build without a valid Steam AppID configuration or without Steam running in the background, Steamworks initialization will fail. Ensure you have the proper `steam_appid.txt` if testing locally.
- **VR headset not detected:**
  Ensure your OpenXR runtime is set correctly (e.g., SteamVR or Oculus) and that the headset is active *before* launching the game.
:::

## Related Pages
* [VR Systems](../vr/index.md)
* [Exporting Standalone](../../publishing/exporting-standalone.md)
* [Services](../services/index.md)
