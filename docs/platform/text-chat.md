---
title: "Text Chat"
icon: "💬"
created: 2026-05-20
updated: 2026-11-11
---

# Text Chat

Built-in text chat for multiplayer games. Messages are routed through the host, validated, filtered, and displayed via the standard overlay UI - no game code required.

The API lives in the `Sandbox.Platform` namespace.

## Project Settings

Chat can be configured in **Project Settings > Platform**:

- **Chat Enabled** - toggle the entire chat system on or off
- **Show Chat UI** - hide the built-in overlay while still processing messages and firing events (useful for custom chat UIs)
- **Max Message Length** - cap message length (up to 256 characters)

## Sending Messages

### `Chat.Say( string message )`

Send a chat message to the server. The host validates and broadcasts it to all connected players.

```csharp
Chat.Say( "Hello everyone!" );
```

### `Chat.AddText( string message )`

Add a local-only notification to the chat feed. Not networked.

```csharp
Chat.AddText( $"{player.Name} joined the game" );
```

:::info
Messages are automatically sanitized, rate-limited, and passed through Steam's text filter. Blocked users are filtered out on the receiving end.
:::

## Scene Events

Implement `IChatEvent` on a component to intercept, modify, suppress, or filter messages. This event fires in two places:

- **On the host** - before broadcasting. You can modify the message, suppress it, or set a `RecipientFilter`.
- **On receiving clients** - after delivery. You can still suppress the message to prevent it from showing in the UI.

```csharp
public class TeamChat : Component, IChatEvent
{
    public void OnChatMessage( ChatMessageEvent e )
    {
        if ( !e.Message.StartsWith( "/team" ) )
            return;

        // Strip the command prefix
        e.Message = e.Message["/team ".Length..];

        // Only deliver to players on the same team
        var senderTeam = GetTeam( e.Sender );
        e.RecipientFilter = connection => GetTeam( connection ) == senderTeam;
    }
}
```

### `ChatMessageEvent`

| Property | Type | Description |
|----------|------|-------------|
| `Message` | `string` | The message text (mutable) |
| `Sender` | `Connection` | Who sent it (`null` for system messages) |
| `Suppress` | `bool` | Set `true` to prevent delivery |
| `RecipientFilter` | `Func<Connection, bool>` | Per-connection visibility (host only) |
