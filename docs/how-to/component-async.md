---
title: "Use Async Tasks"
icon: "🧵"
created: 2023-12-28
updated: 2024-10-03
sources:
  - engine/Sandbox.Engine/Scene/Components/Component.cs
---

# Use Async Tasks

Async tasks allow your game code to wait over time—like pausing for a few seconds or waiting for an HTTP request to finish—without freezing the main game loop.

## Quick Working Example

```csharp
using Sandbox;
using System.Threading.Tasks;

public sealed class AsyncSpawner : Component
{
    [Property] public GameObject PrefabToSpawn { get; set; }

    protected override void OnStart()
    {
        // Start the async sequence without blocking the game
        _ = SpawnAfterDelay( 2.0f );
    }

    async Task SpawnAfterDelay( float waitSeconds )
    {
        // Wait for the specified amount of time
        await Task.DelaySeconds( waitSeconds );

        // Then execute the next line
        PrefabToSpawn.Clone( GameObject.WorldPosition );
    }
}
```

### Animating Properties over Time
You can use a loop with `await Task.Frame()` to smoothly change values over multiple frames. This is great for simple procedural animations.

```csharp
async Task LerpSize( float seconds, Vector3 to, Easing.Function easer )
{
	TimeSince timeSince = 0;
	Vector3 from = GameObject.WorldScale;
 
	while ( timeSince < seconds )
	{
		var size = Vector3.Lerp( from, to, easer( timeSince / seconds ) );
		GameObject.WorldScale = size;
		
		await Task.Frame(); // Wait one frame before continuing the loop
	}
}

protected override void OnStart()
{
	_ = PerformGrowSequence();
}

async Task PerformGrowSequence()
{
	await LerpSize( 3.0f, Vector3.One * 3.3f, Easing.BounceOut );
	await LerpSize( 1.0f, Vector3.One * 4.0f, Easing.EaseInOut );
	await LerpSize( 1.0f, Vector3.One * 3.0f, Easing.EaseInOut );
}
```

### Awaiting Multiple Tasks
You can wait for multiple separate async tasks to finish simultaneously using `Task.WhenAll`. This feels like multithreading, but it's safely running on the main thread.

```csharp
async Task DoMultipleThings()
{
	// Start both tasks immediately, notice no await here
	Task taskOne = Task.DelaySeconds( 2.0f );
	Task taskTwo = Task.DelaySeconds( 3.0f );

	// Pause here until BOTH tasks have completed
	await Task.WhenAll( taskOne, taskTwo );
	
	Log.Info("Both timers are done!");
}
```

### Returning Values from Async
Async Tasks can return values. For example, fetching data from the internet takes an unpredictable amount of time, so it must be async.

```csharp
async Task<string> GetKanyeQuote()
{
	string kanyeQuote = await Http.RequestStringAsync( "https://api.kanye.rest/" );
	return kanyeQuote;
}

async Task PrintKanyeQuote()
{
	string quote = await GetKanyeQuote();
	Log.Info( $"KANYE SAID: {quote}" );
}
```

### Calling Async from Sync Code
To start an async task from a synchronous method (like `OnUpdate` or `OnStart`), simply call it without `await`. Using the discard variable `_` tells the compiler you intentionally aren't waiting for the result.

```csharp
protected override void OnStart()
{
	// Start the task and immediately move on
	_ = PrintKanyeQuote();
}
```

If you want to use the result of an async task in a synchronous method without awaiting, you can use `.ContinueWith()`.

```csharp
protected override void OnStart()
{
    // Will run async and run the Action inside ContinueWith when it finishes
    GetKanyeQuote().ContinueWith( task => Log.Info( $"Kanye: {task.Result}" ) );
}
```

## Troubleshooting

:::danger Task Cancelled Exception
If a GameObject is destroyed while one of its Component's async tasks is waiting (like during `Task.DelaySeconds`), the engine will automatically cancel the task and throw a `TaskCanceledException`. This is a safety feature to prevent code running on destroyed objects. If you need to catch this, wrap your `await` in a try/catch block.
:::

:::warning Tasks Stacking Up
A common error is letting tasks stack up uncontrollably. For example, if a user clicks a button that starts a 2-second reload animation via async, and they click it 10 times, you now have 10 reload sequences running over each other. Use a boolean flag (e.g., `bool _isReloading`) to prevent starting a new task if one is already running.
:::

:::warning Running While Disabled
By default, an async method isn't guaranteed to stop just because its Component or GameObject was disabled. It only auto-cancels if the object is *destroyed*. Be considerate of this when writing long-running loops.
:::

## Related Pages
- [Http Requests](../systems/networking-multiplayer/http-requests.md)
