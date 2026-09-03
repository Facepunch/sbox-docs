---
title: "Stats"
icon: "📉"
created: 2024-08-23
updated: 2026-09-03
---

# Stats

Your game can record stats for each player that plays it. Here are some examples of things stats can record.

* Number of zombies killed
* Fastest time
* Total meters walked
* Coins collected
* Longest time
* Bullets fired
* Kills per weapon

You can query and display these stats, or use them in any other way you want. You can query stats from individual players, and you can get the compound stats globally.

## Defining Stats

You define your stats on your game package's page on sbox.game, under `Services > Stats`. That's where a stat's title, description and unit come from, which is what you get back in `Title`, `Description` and `Unit` when you read it.

A stat doesn't strictly have to be defined to work. The name you pass to `Increment` or `SetValue` is just a string, and it will be recorded either way - you can even build a [leaderboard](/services/leaderboards/index.md) from a name you never registered. Defining it is what gives you the display info, so do it for anything you intend to show to players.

# Recording Stats

Depending on what you're doing, you either want to increment your stat..

```csharp
public void OnZombieKilled()
{
    Sandbox.Services.Stats.Increment( "zombies_killed", 1 );
}
```

Or just set it directly..

```csharp
public void OnGameFinished()
{
    Sandbox.Services.Stats.Increment( "wins", 1 );
    Sandbox.Services.Stats.SetValue( "win-time", SecondsTaken );
}
```


:::info
You can call these apis as often as you like. We batch the stats and send them when we're ready.

:::

## Attaching Data

Both `Increment` and `SetValue` take optional extra data, which is available when querying leaderboards. Pass a `Dictionary<string, object>` (`SetValue` also accepts any object as its `data` argument - it gets serialized to JSON and flattened into one).

```csharp
Sandbox.Services.Stats.SetValue( "win-time", SecondsTaken, new Dictionary<string, object>
{
    { "map", Game.ActiveScene.Title },
    { "difficulty", "hard" }
} );
```

## Flushing

Stats are batched and sent when we're ready, which is usually what you want. If you need them submitted at a specific moment - the end of a round, or before you show a leaderboard you just contributed to - you can flush manually.

```csharp
// Fire and forget
Sandbox.Services.Stats.Flush();

// Or wait for the batch to be sent
await Sandbox.Services.Stats.FlushAsync();

// Wait for the batch to be sent AND processed by the backend
await Sandbox.Services.Stats.FlushAndWaitAsync();
```

Use `FlushAndWaitAsync` when you're about to read the value back, since `FlushAsync` only guarantees it left the client.

# Reading Stats

Stats are available in two forms. Either global, or player.

```csharp
// get global zombie kill count
var zombies = Sandbox.Services.Stats.Global.Get( "zombies_killed" );

Log.Info( $"there have been {zombies.Sum} zombies killed by {zombies.Players} players!" );
```

```csharp
// Get local player zombie kill count
var zombies = Sandbox.Services.Stats.LocalPlayer.Get( "zombies_killed" );

Log.Info( $"You have killed {zombies.Sum} zombies!" );
```

```csharp
// Get stats for Garry in SS1
var stats = Sandbox.Services.Stats.GetPlayerStats( "facepunch.ss1", 76561197960279927 );

// wait for them to download
await stats.Refresh();

var zombies = stats.Get( "zombies_killed" );

Log.Info( $"Garry has killed {zombies.Sum} zombies!" );
```

A stat isn't a single number, it's an aggregate. Both player and global stats give you:

| Property | Description |
|----------|-------------|
| `Sum` | All submitted values added together |
| `Min` / `Max` | Lowest and highest single value submitted |
| `Avg` | Mean of every value submitted |
| `Value` | The value for the stat's own aggregation, as configured |
| `ValueString` | `Value` formatted for display, including the unit |
| `Title` / `Description` / `Unit` | Display info, from the stat's definition on the package |

Player stats also have `Last` and `LastValue`, the timestamp and value of the most recent submission. Global stats instead have `Players` (how many players have contributed) and `Velocity` (how fast the stat is currently moving).

Both collections are enumerable and indexable, so you can list everything without knowing the names up front:

```csharp
foreach ( var stat in Sandbox.Services.Stats.LocalPlayer )
{
    Log.Info( $"{stat.Title}: {stat.ValueString}" );
}

var kills = Sandbox.Services.Stats.LocalPlayer["zombies_killed"];
```

Use `TryGet` if you're not sure a stat exists yet - `Get` on an unknown name gives you a default stat rather than throwing.

## Map Stats

Maps are packages too, so a map can record its own stats separately from the game running it. `Stats.Map` mirrors the API against whichever map package is currently loaded.

```csharp
Sandbox.Services.Stats.Map.SetValue( "secrets_found", 1 );

var secrets = Sandbox.Services.Stats.Map.GetLocal( "secrets_found" );
var global = Sandbox.Services.Stats.Map.GetGlobal( "secrets_found" );
```

`Local` and `Global` give you the whole collections, the same `PlayerStats` and `GlobalStats` types described above.

:::warning
There's no `Stats.Map.Increment`, and despite its name `Stats.Map.SetValue` adds to the stat rather than overwriting it - it behaves like `Increment`.
:::
