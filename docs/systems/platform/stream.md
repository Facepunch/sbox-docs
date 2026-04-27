---
title: "Stream Integration"
sources:
  - engine/Sandbox.Engine/Platform/Stream/Streamer.cs
  - engine/Sandbox.Engine/Platform/Stream/StreamUser.cs
updated: 2026-04-25
created: 2026-04-27
---

# Stream Integration

The Stream API exposes whichever streaming platform the local user has linked (currently Twitch) as a small set of static methods on `Streamer`. You can read who's streaming, how many viewers they have, send messages into chat, and look up users.

> The current API surface is intentionally small. Chat-event subscriptions, poll/prediction hooks, and per-message subscriber checks are **not** part of the public C# API on `Streamer` today — if you need those, query Twitch's REST API directly via `Sandbox.Http` (see [HTTP Requests](../networking-multiplayer/http-requests.md)) or set up an external relay.

## Checking the local stream state

```csharp
using Sandbox;

public sealed class StreamHud : Component
{
    protected override void OnStart()
    {
        if ( !Streamer.IsActive )
        {
            Log.Info( "User isn't streaming." );
            return;
        }

        Log.Info( $"{Streamer.Username} is live on {Streamer.Service}" );
        Log.Info( $"Title: {Streamer.Title}" );
        Log.Info( $"Game: {Streamer.Game}" );
        Log.Info( $"Viewers: {Streamer.ViewerCount}" );
    }
}
```

`Streamer.Service` is a `StreamService` enum (e.g. `Twitch`); `IsActive` is true only while the user is actually live.

## Looking up a user

`GetUser` returns a `StreamUser` for a given username, or for the streamer themself if you pass `null`.

```csharp
public async Task PrintFollower( string username )
{
    var user = await Streamer.GetUser( username );
    if ( user is null ) return;

    Log.Info( $"{user.DisplayName} — {user.Description}" );
}
```

The `StreamUser` struct exposes the basic profile (display name, description, avatar URL). It does not currently include subscriber/follower flags — request those over HTTP if you need them.

## Sending messages and moderating

```csharp
Streamer.SendMessage( "Welcome to the stream!" );
Streamer.ClearChat();

// Moderation — the streamer must be the one running the build
Streamer.BanUser( "spambot42", "spam", duration: 600 );  // 10-minute timeout
Streamer.UnbanUser( "redeemed_user" );
```

`BanUser` with `duration: 0` is a permanent ban; any positive value is a timeout in seconds.

## Stream metadata you can change

The streamer's title, game, and language are exposed as settable properties:

```csharp
Streamer.Title    = "Speedrunning my own game";
Streamer.Game     = "s&box";
Streamer.Language = "en";
Streamer.Delay    = 30;          // stream delay in seconds
```

## Troubleshooting

:::warning Not active
Most members of `Streamer` only do something useful while `Streamer.IsActive` is true. Always check it first; otherwise `Username` is empty, `ViewerCount` is 0, and `SendMessage` is a no-op.
:::

:::info Where to put platform-specific event handling
If you need real-time chat events, polls, channel-point redemptions, or other Twitch-specific hooks, run a small WebSocket connection to Twitch from your game using `Sandbox.WebSocket`. The engine intentionally doesn't try to model every platform-specific event.
:::

## Related Pages
* [Platform Integration](index.md)
* [Twitch](twitch.md)
* [HTTP Requests](../networking-multiplayer/http-requests.md)
* [WebSockets](../networking-multiplayer/websockets.md)
