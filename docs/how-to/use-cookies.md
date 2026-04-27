---
title: "Use Cookies"
icon: "🍪"
sources:
  - engine/Sandbox.Engine/Systems/Cookies/Cookie.cs
  - engine/Sandbox.Engine/GlobalNamespace.cs
created: 2026-04-27
updated: 2026-04-27
---

# Use Cookies

Learn how to use Game.Cookies to store and retrieve lightweight data, like user preferences, across sessions.

## Quick Working Example

```csharp
using Sandbox;

public sealed class SettingsManager : Component
{
    // A simple property we want to save
    [Property] public bool ShowSubtitles { get; set; } = true;

    protected override void OnStart()
    {
        // 1. Load the value from cookies when the component starts.
        // The second parameter is the default value if the cookie doesn't exist.
        ShowSubtitles = Game.Cookies.Get<bool>( "settings.show_subtitles", true );
        
        Log.Info( $"Show Subtitles is set to: {ShowSubtitles}" );
    }

    public void ToggleSubtitles()
    {
        ShowSubtitles = !ShowSubtitles;
        
        // 2. Save the new value back to cookies.
        Game.Cookies.Set( "settings.show_subtitles", ShowSubtitles );
        
        Log.Info( "Subtitles toggled and saved." );
    }
}
```

### Storing and Retrieving Strings

If you just need to save a string (like a player's last entered lobby name), you can use the dedicated string methods.

```csharp
// Save a string
Game.Cookies.SetString( "last_lobby_name", "My Cool Server" );

// Retrieve a string, providing a fallback if it doesn't exist
string lobbyName = Game.Cookies.GetString( "last_lobby_name", "Default Lobby" );

// Try to get a string, returning a boolean if successful
if ( Game.Cookies.TryGetString( "last_lobby_name", out var savedName ) )
{
    Log.Info( $"Found saved lobby: {savedName}" );
}
```

### Storing Custom Objects

You can also store small C# objects. The cookie system will automatically serialize them to JSON.

```csharp
public class PlayerStats
{
    public int MatchesPlayed { get; set; }
    public int HighScore { get; set; }
}

public void SaveStats( PlayerStats stats )
{
    // Automatically serialized to JSON
    Game.Cookies.Set( "player_stats", stats );
}

public PlayerStats LoadStats()
{
    // Retrieve the object, or return a new one if it doesn't exist
    return Game.Cookies.Get<PlayerStats>( "player_stats", new PlayerStats() );
}
```

### Removing Cookies

If a setting is reset to default or no longer needed, you can remove it from the cache to keep the save file clean.

```csharp
Game.Cookies.Remove( "settings.show_subtitles" );
```

## Configuration

The `Game.Cookies` system provides a few methods for retrieving and setting data:

| Method | Description |
|---|---|
| `SetString( string key, string value )` | Sets a string cookie. |
| `GetString( string key, string fallback )` | Gets a string cookie, returning the fallback if it doesn't exist. |
| `TryGetString( string key, out string val )` | Attempts to get a string cookie, returning true if successful. |
| `Set<T>( string key, T value )` | Sets a cookie of type `T`, serializing it to JSON. |
| `Get<T>( string key, T fallback )` | Gets a cookie of type `T`, returning the fallback if it doesn't exist. |
| `TryGet<T>( string key, out T val )` | Attempts to get a cookie of type `T`, returning true if successful. |
| `Remove( string key )` | Removes the cookie with the specified key. |

## Troubleshooting

:::danger Do not use Cookies for critical save data!
By default, cookies expire 30 days after they are last accessed. If a player stops playing your game for a month, their cookies will be deleted. **Never** use `Game.Cookies` to save the player's level, inventory, or actual game progress. Use `FileSystem.Data` (see [Save and Load Data](save-and-load.md)) for permanent saves.
:::

:::warning Complex Objects Failing to Serialize
When using `Set<T>`, ensure your class is a simple data object (POCO) with public **Properties** (`{ get; set; }`). Fields are not serialized by default. Do not attempt to store `GameObject` or `Component` references in a cookie.
:::

## Related Pages
- [Save and Load Data](save-and-load.md)
