---
title: "Async Component Loading"
icon: "⏳"
sources:
  - engine/Sandbox.Engine/Scene/Components/Component.Loading.cs
created: 2026-04-27
updated: 2026-04-27
---

# Async Component Loading

Use the `OnLoad` component lifecycle hook to perform asynchronous operations, such as downloading assets or generating procedural data, before a scene starts running.

## Quick Working Example

```csharp
using Sandbox;
using System.Threading.Tasks;

public sealed class LevelGenerator : Component
{
	// Override OnLoad to perform work before the scene starts
	protected override async Task OnLoad()
	{
		Log.Info("Starting procedural generation...");

		// Simulate heavy work
		await Task.DelayRealtimeSeconds( 2.0f );

		Log.Info("Procedural generation complete!");
	}
}
```

### Providing Loading Progress

If you have a very long loading task, you might want to report progress back to the engine's loading screen so the player knows the game hasn't frozen. You can do this by overriding the `OnLoad( LoadingContext )` variant.

```csharp
using Sandbox;
using System.Threading.Tasks;

public sealed class AssetDownloader : Component
{
	protected override async Task OnLoad( LoadingContext context )
	{
		// Tell the loading screen what we are doing
		context.Title = "Downloading Custom Assets...";
		context.Progress = 0.0f;

		for ( int i = 0; i < 100; i++ )
		{
			// Simulate downloading a file
			await Task.DelayRealtimeSeconds( 0.05f );

			// Update the progress bar (0.0 to 1.0)
			context.Progress = (i + 1) / 100f;
		}

		Log.Info("Assets downloaded.");
	}
}
```

### Waiting for Sub-Systems

You might have a manager component that needs to initialize a database or connection before other components can safely use it. 

```csharp
public sealed class DatabaseManager : Component
{
	public bool IsReady { get; private set; }

	protected override async Task OnLoad()
	{
		// Simulate connecting to a service
		await Task.DelayRealtimeSeconds( 1.0f );
		IsReady = true;
	}
}
```

## Troubleshooting

:::danger Unbounded or Infinite Tasks
If your `OnLoad` task gets stuck in an infinite loop or waits on a network request that never times out, the game will hang indefinitely on the loading screen. Always ensure your async operations have proper timeouts or cancellation logic.
:::

:::warning Not for Gameplay Logic
Do not start spawning gameplay enemies or moving players during `OnLoad`. The scene is still initializing, and `OnStart` hasn't been called yet. Use `OnLoad` strictly for preparing data and assets. Use `OnStart` for gameplay initialization.
:::

## Related Pages
* [Component Lifecycle](../scene/components/component-lifecycle.md)
* [Use Async Tasks](component-async.md)
