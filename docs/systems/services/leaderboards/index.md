---
title: "Leaderboards"
icon: "🏆"
created: 2024-08-23
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Game/Services/Leaderboards/Leaderboards.cs
---

# Leaderboards

Leaderboards are just stats, aggregated and ordered, allowing you to rank players globally or against their friends.

## Quick Working Code Example

```csharp
public class LeaderboardViewer : Component
{
	protected override async Task OnStart()
	{
		// Get a leaderboard to show who has killed the most zombies globally
		var board = Sandbox.Services.Leaderboards.GetFromStat( "facepunch.ss1", "zombies_killed" );
		await board.Refresh();

		foreach ( var entry in board.Entries )
		{
			Log.Info( $"{entry.Rank} - {entry.DisplayName} - {entry.Value}" );
		}
	}
}
```

### Filtering By Country

You can filter a leaderboard by country. This will only show stats that were created in that country.

```csharp
var board = Sandbox.Services.Leaderboards.GetFromStat( "facepunch.ss1", "zombies_killed" );
board.SetCountryCode( "gb" );
await board.Refresh();

foreach ( var entry in board.Entries )
{
	Log.Info( $"{entry.Rank} - {entry.DisplayName} - {entry.Value} [{entry.CountryCode}]" );
}
```

:::tip
If you pass the country code as `"auto"`, it will use the current player's location.
:::

### Filtering By Date

You can filter leaderboards by date, allowing for yearly, weekly, monthly, or daily leaderboards.

```csharp
var board = Sandbox.Services.Leaderboards.GetFromStat( "facepunch.ss1", "zombies_killed" );
board.FilterByMonth();
board.SetDatePeriod( new System.DateTime( 2024, 8, 1 ) );
await board.Refresh();

foreach ( var entry in board.Entries )
{
	Log.Info( $"{entry.Rank} - {entry.DisplayName} - {entry.Value} [{entry.CountryCode}]" );
}
```

:::tip
If you don't set a date period, it will use the current date.
:::

### Centering the Leaderboard

You can focus the leaderboard on a certain player. This will show the results around that player. It is often nicer to show a player's contemporaries rather than showing the top 20 all the time.

```csharp
var board = Sandbox.Services.Leaderboards.GetFromStat( "facepunch.ss1", "zombies_killed" );
board.CenterOnSteamId( 76561197960279927 );
await board.Refresh();

foreach ( var entry in board.Entries )
{
	Log.Info( $"{entry.Rank} - {entry.DisplayName} - {entry.Value}" );
}
```

:::tip
You can also call `.CenterOnMe()` to center the leaderboard on the local player.
:::

### Aggregation and Sorting

Instead of the sum of a stat, you might want to show people's lowest or highest single result. For example, to show a list of the fastest win times:

```csharp
var board = Sandbox.Services.Leaderboards.GetFromStat( "facepunch.ss1", "victory_elapsed_time" );

board.SetAggregationMin(); // select the lowest value from each player
board.SetSortAscending();  // order by the lowest value first
board.FilterByMonth();     // only show results from this month
board.CenterOnMe();        // offset so I'm in the middle of the results
board.MaxEntries = 100;    // fetch up to 100 entries

await board.Refresh();

foreach ( var entry in board.Entries )
{
	Log.Info( $"{entry.Rank} - {entry.DisplayName} - {entry.Value} [{entry.Timestamp}]" );
}
```

:::info
You can aggregate by `Sum`, `Min`, `Max`, `Avg`, or `Last`.
:::

:::info
The entry timestamp holds the time of the selected result.
:::

## Troubleshooting

:::danger "No entries in my leaderboard"
Double check that the stat string matches perfectly what you configured. Make sure you actually submitted stats to this package. Leaderboards only update when the underlying stats receive data!
:::

## Related Pages
* [Stats](../stats.md)
* [Achievements](../achievements.md)