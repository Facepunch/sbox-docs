---
title: "Steam Integration"
sources:
  - engine/Sandbox.Engine/Platform/Steam/
created: 2026-04-27
updated: 2026-04-27
---

# Steam Integration

The s&box engine natively integrates with Steam through the Steamworks API, giving you access to friends lists, rich presence, matchmaking, and networking.

## Quick Start

```csharp
using Sandbox;
using Sandbox.Engine.Platform.Steam;

public sealed class SteamFriendsComponent : Component
{
    protected override void OnStart()
    {
        // Only run if we are actually connected to Steam
        if ( !SteamClient.IsValid )
            return;

        var myName = SteamClient.Name;
        Log.Info( $"Connected to Steam as {myName}" );
        
        // Let's see who is playing the same game
        foreach ( var friend in SteamFriends.GetFriends() )
        {
            if ( friend.IsPlayingThisGame )
            {
                Log.Info( $"{friend.Name} is also playing!" );
            }
        }
    }
}
```

## Core Features

### Friends and Rich Presence
You can use `SteamFriends` to access the local user's friends list, check their online status, and set Rich Presence information (which appears in the Steam Friends list when someone clicks "View Game Info").

### Matchmaking
The `SteamMatchmaking` class allows you to create, find, and join Steam lobbies. Lobbies are the primary way players group up before starting a networked game.

### User Information
`SteamClient` provides details about the local user, including their Steam ID, display name, and avatar.

## Common Patterns

### Setting Rich Presence
A common task is setting the Rich Presence string so friends can see what the player is doing.

```csharp
public sealed class GameStateNotifier : Component
{
    public void SetInGamePresence( string mapName )
    {
        if ( SteamClient.IsValid )
        {
            SteamFriends.SetRichPresence( "status", $"Playing on {mapName}" );
        }
    }
}
```

## Troubleshooting

:::warning Initialization Failure
If `SteamClient.IsValid` is false, it typically means Steam is not running on the local machine, or the `steam_appid.txt` file is missing/incorrect in a Standalone build. Ensure Steam is running and you are logged in.
:::

## Related Pages
* [Platform Integration](./index.md)
* [Networking & Multiplayer](../networking-multiplayer/index.md)
