---
title: "Voice Chat"
icon: "🎙️"
created: 2026-11-11
updated: 2026-11-11
---

# Voice Chat

Players can talk to each other in any multiplayer game, with no setup. Everyone hears everyone, and there's a built-in list of who's talking.

Players choose push to talk or open microphone in their own settings, under **Settings > Audio**.

The API lives in the `Sandbox.Platform` namespace.

## Project Settings

Voice chat can be configured in **Project Settings > Platform**:

- **Enabled** - lets everyone talk to everyone. Turn it off and nobody's heard unless your game uses `Voice` components.
- **Show Speaker List** - the built-in list of who's talking. Turn it off if you show your own. The microphone indicator always shows.

## Voice Components

If you want proximity voice, teams, or lip sync, add a `Voice` component to your player prefab.

Once your scene has any `Voice` components, everyone-hears-everyone stops. Only players talking through their own `Voice` are heard, and the built-in speaker list goes away, so show your own.

- **Disable a player's `Voice`** to mute them.
- **World Space Playback**, **Distance** and **Falloff** set how voice carries through the world.
- **Push To Talk Input** is the key they hold, for players using push to talk.
- **Renderer** drives a model's mouth while they talk.

### Who hears who

Make your own `Voice` and override who it's sent to:

```csharp
public sealed class TeamVoice : Voice
{
	protected override IEnumerable<Connection> ExcludeFilter()
	{
		return Connection.All.Where( x => GetTeam( x ) != GetTeam( Connection.Local ) );
	}
}
```

Override `ShouldHearVoice( Connection )` to ignore someone on the receiving end instead.

## Who's Talking

Use these for your own speaker list, or a talking icon on your scoreboard. They work with or without `Voice` components.

```csharp
if ( VoiceChat.IsTalking( connection ) )
{
	// Draw a speaker icon
}

// How loud they are, in decibels. -96 when they're not talking.
var level = VoiceChat.GetLevel( connection );
```
