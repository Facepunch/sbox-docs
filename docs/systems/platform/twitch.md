---
title: "Twitch Integration"
sources:
  - engine/Sandbox.Engine/Platform/Stream/Twitch/
  - engine/Sandbox.Engine/Platform/Stream/Streamer.cs
updated: 2026-04-25
created: 2026-04-27
---

# Twitch Integration

The Twitch implementation backs the platform-agnostic [`Streamer`](stream.md) API. When the user has linked their Twitch account through s&box and is broadcasting, `Streamer.Service` reports `StreamService.Twitch` and the rest of the API works against the Twitch backend.

This page covers what's Twitch-specific and how to extend the integration when the static `Streamer` API doesn't cover your use case.

## Knowing you're on Twitch

```csharp
if ( Streamer.IsActive && Streamer.Service == StreamService.Twitch )
{
    // Twitch-specific logic
}
```

Read [Stream Integration](stream.md) for the platform-agnostic API surface (sending messages, viewer count, looking up users, moderating).

## Going beyond the static API

The `Streamer` class deliberately exposes only a small surface — basics that work across providers. For Twitch-specific features that aren't in the public C# API today (real-time chat events, polls, predictions, channel-point redemptions, follower notifications), you have two options:

**1. Talk to Twitch directly via HTTP / WebSocket.**

Use [`Sandbox.Http`](../networking-multiplayer/http-requests.md) for REST calls and [`Sandbox.WebSocket`](../networking-multiplayer/websockets.md) for the EventSub WebSocket. You provide your own app credentials. This is the most flexible path and Twitch-supported.

**2. Run an external relay.**

Stand up a small server (Node, .NET, whatever) that subscribes to Twitch EventSub and pushes events into your game over a private WebSocket. Useful when you want one server brokering events for multiple game clients.

## What Twitch sends in users you query

`Streamer.GetUser(username)` calls into the Twitch user lookup and returns a `StreamUser`. Twitch populates the basic profile fields (`DisplayName`, `Description`, avatar URL); subscriber/VIP/moderator status is **not** part of `StreamUser` today — query Twitch's REST API directly for those flags.

## Troubleshooting

:::warning Not authenticated
If `Streamer.IsActive` is true but calls return empty data, the user's Twitch token may have expired. They need to re-link the account in s&box settings.
:::

:::warning Rate limits
Twitch enforces request rate limits per app credential. If you're polling user data or chat-replay aggressively, batch your requests and cache results. Real-time event feeds (EventSub WebSocket) are not rate-limited the same way as REST polls.
:::

## Related Pages
* [Stream Integration](stream.md) — the platform-agnostic API surface
* [Platform Integration](index.md)
* [HTTP Requests](../networking-multiplayer/http-requests.md)
* [WebSockets](../networking-multiplayer/websockets.md)
