---
title: "Stats"
icon: "📉"
created: 2024-08-23
updated: 2024-09-09
sources:
  - engine/Sandbox.Engine/Game/Services/Stats/Stats.cs
---

# Stats

Your game can record stats for each player that plays it, allowing you to track player actions and submit data to leaderboards.

## Quick Working Example

```csharp
public class ZombieGameMode : Component
{
	public void OnZombieKilled()
	{
		// Increment the global "zombies-killed" stat by 1
		Sandbox.Services.Stats.Increment( "zombies-killed", 1 );
	}

	public void OnGameFinished( float secondsTaken )
	{
		// Set a specific value for the completion time
		Sandbox.Services.Stats.SetValue( "win-time", secondsTaken );
	}
}
```

### Recording Stats

Depending on what you're doing, you either want to increment your stat or just set it directly.

```csharp
public void OnGameFinished()
{
	Sandbox.Services.Stats.Increment( "wins", 1 );
	Sandbox.Services.Stats.SetValue( "win-time", SecondsTaken );
}
```

:::info
You can call these APIs as often as you like. The engine batches the stats and sends them automatically when ready.
:::

You can also attach extra contextual data when calling `SetValue` or `Increment`. This data is passed as a `Dictionary<string, object>` and becomes available when querying leaderboards.

```csharp
var contextData = new Dictionary<string, object>
{
	{ "difficulty", "hard" },
	{ "character", "soldier" }
};

Sandbox.Services.Stats.SetValue( "win-time", SecondsTaken, contextData );
```

### Reading Stats

Stats are available in two forms: global (across all players) or specific to a player.

**Global Stats:**
```csharp
// Get global zombie kill count
var zombies = Sandbox.Services.Stats.Global.Get( "zombies_killed" );

Log.Info( $"There have been {zombies.Sum} zombies killed by {zombies.Players} players!" );
```

**Local Player Stats:**
```csharp
// Get local player zombie kill count
var zombies = Sandbox.Services.Stats.LocalPlayer.Get( "zombies_killed" );

Log.Info( $"You have killed {zombies.Sum} zombies!" );
```

**Specific Player Stats:**
```csharp
// Get stats for Garry in a specific package
var stats = Sandbox.Services.Stats.GetPlayerStats( "facepunch.ss1", 76561197960279927 );

// Wait for them to download
await stats.Refresh();

var zombies = stats.Get( "zombies_killed" );

Log.Info( $"Garry has killed {zombies.Sum} zombies!" );
```

## Troubleshooting

:::warning "Stats not updating immediately"
Because stat changes are batched and sent periodically, they might not instantly reflect if you immediately query the server. Use `Sandbox.Services.Stats.FlushAsync()` if you absolutely must ensure stats are uploaded before proceeding.
:::

## Related Pages
* [Leaderboards](leaderboards/index.md)
* [Achievements](achievements.md)