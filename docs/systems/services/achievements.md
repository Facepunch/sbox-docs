---
title: "Achievements"
icon: "🏅"
created: 2024-09-10
updated: 2024-09-10
sources:
  - engine/Sandbox.Engine/Game/Services/Achievements/Achievements.cs
---

# Achievements

Your game can have multiple achievements for players to unlock, which rewards them with a score and an icon on their profile.

## Quick Working Code Example

```csharp
public class GameProgress : Component
{
	public void OnBossDefeated()
	{
		// Manually unlock an achievement for the local player
		Sandbox.Services.Achievements.Unlock( "boss_defeated" );
	}
	
	public void PrintAllAchievements()
	{
		// List all available achievements in the game
		foreach ( var a in Sandbox.Services.Achievements.All )
		{
			Log.Info( $"Achievement: {a.Title} - {a.Description}" );
		}
	}
}
```

### Stat Based Unlock

When your achievement unlock mode is set to "Stat" on the backend, it will automatically be unlocked for you. All you need to do is make sure the underlying stat is updated properly using the `Stats` API.

An example of a stat based achievement would be "100 coins collected". You would do `Sandbox.Services.Stats.Increment( "coins", 1 )` in your code every time you collected a coin.

**Example Setup on Backend:**
* Target Stat: "coins"
* Aggregation: Sum (you want to add the 1's values together)
* Min Value: 0, Max Value: 100 (unlocks at 100)
* Show Progress: true (will show progress between 0 and 100 as a percentage)

### Manual Unlock

If the achievement unlock mode is set to manual, then you can unlock it explicitly in your C# code.

```csharp
Sandbox.Services.Achievements.Unlock( "first_win" );
```

### Listing Achievements

You can get the list of achievements at any time in your game to display them in your own UI.

```csharp
foreach ( var a in Sandbox.Services.Achievements.All )
{
	if ( a.IsUnlocked )
	{
		Log.Info( $"Player has unlocked {a.Title}!" );
	}
}
```

Or from outside the game, you can access it via the web API:

```text
https://services.facepunch.com/sbox/achievement/list?package=facepunch.testbed
```

## Configuration

When configuring achievements on the backend, keep the following in mind regarding icons:
* The icon is automatically resized to 128x128. 
* It will generally be rendered quite small, next to the name of the achievement, so it should be treated more like a clear icon than a detailed picture.

## Troubleshooting

:::danger Achievement limit
The combined score of all achievements in your game cannot exceed 1000. If you try to add an achievement that exceeds this limit, it will not be saved.
:::

:::warning "My achievement isn't unlocking!"
Double-check that the achievement identifier used in `Sandbox.Services.Achievements.Unlock( "achievement_ident" )` perfectly matches the internal name defined on the backend, which is case-sensitive.
:::

## Related Pages
* [Stats](stats.md)
* [Leaderboards](leaderboards/index.md)