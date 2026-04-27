---
title: "GameObjectSystem"
icon: "⚙️"
created: 2023-12-01
updated: 2026-04-11
sources:
  - engine/Sandbox.Engine/Scene/GameObjectSystem/GameObjectSystem.cs
---

# GameObjectSystem

A `GameObjectSystem` allows you to create a system that exists globally in every scene, hooks into precise stages of the scene's lifecycle, and updates code deterministically independent of individual components.

## Quick Working Example

```csharp
public class MyGameSystem : GameObjectSystem
{
	public MyGameSystem( Scene scene ) : base( scene )
	{
		// Listen to a stage executing at order 10
		Listen( Stage.PhysicsStep, 10, DoSomething, "DoingSomething" );
	}

	void DoSomething()
	{
		Log.Info( "Physics Step is happening!" );
	}
}
```

### Accessing Systems

You can retrieve the instance of your system running in the current scene using `Scene.GetSystem<T>()` or `Scene.GetAllComponents<T>()` if interacting with components en masse.

```csharp
var mySystem = Scene.GetSystem<MyGameSystem>();
mySystem.DoSomething();
```

## Configuration

Systems are automatically instantiated. You can listen to specific execution stages in the constructor using `Listen( Stage, Order, Action, DebugName )`.

**Order:** Defines the execution priority if multiple systems listen to the same stage. Smaller numbers run first. A default action might happen at `0`. To guarantee you run before it, use `-1`. To run after it, use `1`.

## Advanced

### Inheritance with `<T>`

For cleaner access, inherit from `GameObjectSystem<T>`. This provides a static `.Current` property that automatically gets the system from the active scene.

```csharp
public class MyGameSystem : GameObjectSystem<MyGameSystem>
{
	public MyGameSystem( Scene scene ) : base( scene ) { }
 
	public void TriggerGlobalEvent()
	{
		Log.Info( "Global event triggered!" );
	}
}

// Access it from anywhere:
MyGameSystem.Current.TriggerGlobalEvent();
```

### Exposing Properties

Any property marked with `[Property]` on a `GameObjectSystem` will be configurable in the project settings and saved.

## Troubleshooting

:::danger Invalid Return
Make sure your system does not execute heavy blocking calls in its listeners to prevent halting the game loop.
:::

## Related Pages
* [Components Overview](components/index.md)
* [Execution Order](components/execution-order.md)
