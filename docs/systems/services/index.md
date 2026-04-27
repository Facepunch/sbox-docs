---
title: "Services"
icon: "💽"
created: 2024-08-23
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Game/Services/
---

# Services

The Sandbox Services API allows you to integrate your game with online features like player statistics, achievements, leaderboards, and authentication, all managed by the s&box backend without requiring you to host your own database.

## Quick Working Example

You can record stats and unlock achievements with simple static method calls from anywhere in your client code:

```csharp
using Sandbox;
using Sandbox.Services;

public class GameTracker : Component
{
	public void OnPlayerWin()
	{
		// Record a win stat - this automatically queues to send to the backend
		Stats.Increment( "wins", 1 );
		
		// Unlock a related achievement defined on the sbox.game dashboard
		Achievements.Unlock( "first_win" );
	}
}
```

## Common Patterns

1. **Stat Aggregation:** You do not need to read the current stat and add to it. Calling `Stats.Increment` simply tells the server "add 1 to whatever this player currently has".
2. **Dashboard Configuration:** You cannot invent an achievement name in code and expect it to work. You must define the achievement (name, description, icon) on your game's developer dashboard on `sbox.game` first.
3. **Asynchronous Queries:** Fetching leaderboards or player inventories involves network requests. These methods are asynchronous (`Task<T>`) and should be awaited properly so you don't freeze the main thread.

## Troubleshooting

:::warning
- **"Stat not found / Achievement not found":** This error occurs if you try to modify a stat or unlock an achievement that hasn't been defined on the `sbox.game` dashboard for your active project. Ensure your project Ident is correct.
- **"Not Logged In":** If you are running the engine in offline mode or disconnected from Steam, services will silently fail or return default values.
:::

## Navigation Hub

* [**Stats**](stats.md) - Record player data like kills, time played, or coins collected.
* [**Achievements**](achievements.md) - Define and unlock achievements with scores and icons.
* [**Leaderboards**](leaderboards/index.md) - Aggregate stats and rank players globally, by country, or by date.
* [**Auth Tokens**](auth-tokens.md) - Generate and validate tokens to authenticate players on your own custom backend servers.
