---
title: "Execution Order"
icon: "🔃"
created: 2024-04-29
updated: 2025-08-15
sources:
  - engine/Sandbox.Engine/Scene/GameObjectSystem/GameObjectSystem.cs
  - engine/Sandbox.Engine/Scene/Components/Component.cs
---

# Execution Order

Execution order dictates exactly when and in what sequence component lifecycle methods (like `OnUpdate` or `OnFixedUpdate`) are called by the engine during a frame.

## Quick Working Example

If you need strict control over execution order (e.g., ensuring your camera moves *after* your player moves), you should bypass `Component.OnUpdate` and instead use a `GameObjectSystem` to hook into a specific stage of the scene tick.

```csharp
using Sandbox;
using System;

public class CustomCameraSystem : GameObjectSystem
{
	public CustomCameraSystem( Scene scene ) : base( scene )
	{
		// Hook into the 'FinishUpdate' stage, which runs after all regular Component OnUpdates.
		// A negative order (e.g. -1) runs before default hooks at 0; positive (e.g. 1) runs after.
		Listen( Stage.FinishUpdate, 0, LateUpdateCamera, "CustomCameraSystem" );
	}

	private void LateUpdateCamera()
	{
		Log.Info( "Updating camera after all components have finished updating." );
	}
}
```

### Standard Component Lifecycle

For the majority of game logic, you don't need strict execution order. Use standard component lifecycle methods.

```csharp
public class NormalComponent : Component
{
	protected override void OnUpdate()
	{
		// Called every frame, but order relative to other components is not guaranteed.
		GameObject.WorldPosition += Vector3.Up * Time.Delta;
	}
}
```

### Controlling Order with GameObjectSystem

When order matters (e.g., UI updating after physics, cameras updating after player movement), create a `GameObjectSystem`. It always exists in the scene, is hooked into the scene's lifecycle, and lets you specify the exact `GameObjectSystem.Stage` and `order` integer to run your logic.

## Configuration: The Flowchart

The following flow chart shows the order of execution for a Scene during a tick. Component methods are executed simultaneously for all active components during their respective blocks. 

![](./images/flowchart.png)

### Available System Stages

If you are using `GameObjectSystem`, here are the `Stage` enum values you can hook into:

| Stage | Description |
|---|---|
| `StartUpdate` | At the very start of the scene update |
| `UpdateBones` | Bones are worked out |
| `PhysicsStep` | Physics step, called in fixed update |
| `Interpolation` | When transforms are interpolated |
| `FinishUpdate` | At the very end of the scene update |
| `StartFixedUpdate`| Called at the start of fixed update |
| `FinishFixedUpdate`| Called at the end of fixed update |
| `SceneLoaded` | Called after a scene has been loaded |

## Troubleshooting

:::warning Unpredictable Components
You should not rely on the order in which the same callback methods get invoked for different GameObjects, it is not predictable. If you find yourself needing `OnUpdate` to run before another component's `OnUpdate`, use a `GameObjectSystem` instead.
:::

:::danger Unregistering Listeners
When using `GameObjectSystem.Listen()`, the engine automatically tracks the `IDisposable` returned by the hook. When the `GameObjectSystem` is disposed (when the scene is disposed), your listeners are cleaned up automatically.
:::

## Related Pages
* [Component Lifecycle](component-lifecycle.md)
* [GameObjectSystem](../gameobjectsystem.md)