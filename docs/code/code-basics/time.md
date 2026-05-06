---
title: "Time"
icon: "⏱️"
created: 2026-05-03
updated: 2026-05-03
---

# Time

s&box provides a set of utilities for working with time in your game. These let you write things like cooldowns, countdowns, and frame-rate independent movement cleanly and without manual bookkeeping.


# Time.Delta

`Time.Delta` is the number of seconds that have elapsed since the last frame. You should multiply any per-frame movement or change by this to make it **frame-rate independent** - so it behaves the same whether your game is running at 30fps or 300fps.

```csharp
protected override void OnUpdate()
{
	// Move 100 units per second, regardless of frame rate
	WorldPosition += Vector3.Forward * 100f * Time.Delta;
}
```


# Time.Now

`Time.Now` is the total time in seconds since the scene started. You can use it to schedule things or measure durations. `Time.Now` is synchronized across the network.

```csharp
float startTime = Time.Now;

// later...
float elapsed = Time.Now - startTime;
Log.Info( $"It has been {elapsed} seconds" );
```

This works, but s&box has cleaner helpers for this pattern - `TimeSince` and `TimeUntil`.


# TimeSince

`TimeSince` is a struct that automatically tracks how long ago it was set. Assigning `0` resets the timer to now. Reading it as a `float` gives you the number of seconds elapsed.

```csharp
TimeSince lastShot = 0;

protected override void OnUpdate()
{
	if ( Input.Pressed( "attack1" ) && lastShot > 0.5f )
	{
		Shoot();
		lastShot = 0; // reset the timer
	}
}
```

This is much simpler than storing `float lastShotTime = Time.Now` and writing `Time.Now - lastShotTime > 0.5f` everywhere.

You can also compare with `<`, `<=`, `>=`, `>`:

```csharp
TimeSince timeSinceGrounded;

// check if the player has been airborne for more than 0.2 seconds
if ( timeSinceGrounded > 0.2f )
{
	// play coyote time logic
}
```


# TimeUntil

`TimeUntil` counts *down* to a moment in the future. Assigning a number of seconds sets the countdown. It implicitly converts to a  `true` `bool` when the time has expired.

```csharp
TimeUntil respawnAt;

void OnDeath()
{
	respawnAt = 5f; // respawn in 5 seconds
}

protected override void OnUpdate()
{
	if ( respawnAt )
	{
		Respawn();
	}
}
```

You can also read it as a `float` to get the **remaining** seconds:

```csharp
float secondsLeft = respawnAt;
Log.Info( $"Respawning in {secondsLeft:F1}s" );
```

### Fraction

`TimeUntil.Fraction` returns a value from `0` (just started) to `1` (expired), which is great for progress bars or lerping effects:

```csharp
TimeUntil reloadDone;

void StartReload()
{
	reloadDone = 2.0f; // 2 second reload
}

protected override void OnUpdate()
{
	float progress = reloadDone.Fraction; // 0 → 1 as time passes
	ReloadBar.SetProgress( progress );
}
```


# RealTimeSince and RealTimeUntil

These work exactly like `TimeSince` and `TimeUntil`, but they use **wall-clock time** (`RealTime.Now`) instead of game time (`Time.Now`).

Use `RealTimeSince` / `RealTimeUntil` when you want the timer to keep ticking even if the game is paused or the time scale is changed (e.g., for UI animations, network timeouts, or cooldowns that should be unaffected by slow motion).

```csharp
RealTimeSince lastHeartbeat = 0;

protected override void OnUpdate()
{
	if ( lastHeartbeat > 30f )
	{
		SendHeartbeat();
		lastHeartbeat = 0;
	}
}
```


# Quick Reference

| Type | Resets with | Reads as | Use for |
|------|-------------|----------|---------|
| `TimeSince` | `= 0` | seconds elapsed | cooldowns, durations |
| `TimeUntil` | `= seconds` | seconds remaining (bool when done) | countdowns, delays |
| `RealTimeSince` | `= 0` | seconds elapsed (wall clock) | UI, network, pause-immune timers |
| `RealTimeUntil` | `= seconds` | seconds remaining (wall clock) | pause-immune countdowns |
| `Time.Delta` | n/a | seconds since last frame | frame-rate independent movement |
| `Time.Now` | n/a | total seconds since scene start | raw time reference |
