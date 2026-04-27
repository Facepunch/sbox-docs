---
title: "Optimize Game Logic & Rendering"
icon: "🔧"
sources:
  - engine/Sandbox.Engine/Scene/GameObject/GameObject.cs
  - engine/Sandbox.Engine/Scene/Components/Component.Update.cs
created: 2026-04-27
updated: 2026-04-27
---

# Optimize Game Logic & Rendering

Use these common strategies to improve your game's framerate and reduce stuttering when you are CPU or GPU bound.

## Quick Working Example

A common source of stutter is instantiating and destroying too many objects, creating garbage collection spikes. Use object pooling instead:

```csharp
using Sandbox;
using System.Collections.Generic;

public sealed class ProjectilePool : Component
{
	[Property] public GameObject ProjectilePrefab { get; set; }
	
	private Queue<GameObject> _pool = new();

	public GameObject GetProjectile( Vector3 position, Rotation rotation )
	{
		GameObject proj;
		
		if ( _pool.Count > 0 )
		{
			// Reuse an existing disabled projectile
			proj = _pool.Dequeue();
			proj.WorldPosition = position;
			proj.WorldRotation = rotation;
			proj.Enabled = true;
		}
		else
		{
			// Pool is empty, create a new one
			proj = ProjectilePrefab.Clone( position, rotation );
		}
		
		return proj;
	}

	public void ReturnProjectile( GameObject proj )
	{
		// Disable and store instead of destroying
		proj.Enabled = false;
		_pool.Enqueue( proj );
	}
}
```

### Caching Component Queries (CPU)

`GameObject.Components.Get<T>()` or `GetAll<T>()` are fast, but if called inside `OnUpdate` every frame by dozens of enemies, the cost adds up. Find the component once and cache it.

```csharp
public sealed class EnemyAI : Component
{
	// Cache the reference here
	private PlayerController _player;

	protected override void OnStart()
	{
		// Expensive query happens ONCE during initialization
		_player = Scene.GetAllComponents<PlayerController>().FirstOrDefault();
	}

	protected override void OnUpdate()
	{
		if ( _player == null ) return;
		
		// Cheap property access happens every frame
		var targetPos = _player.GameObject.WorldPosition;
		// ... move towards target
	}
}
```

### Distance-Based Disabling (CPU)

Don't run logic for objects the player can't see or interact with. You can manually disable components or GameObjects when they are far away.

```csharp
protected override void OnUpdate()
{
	var distToCamera = Vector3.DistanceBetween( GameObject.WorldPosition, Scene.Camera.WorldPosition );
	
	if ( distToCamera > 2000f )
	{
		// We are too far, don't run expensive AI or animation logic
		return; 
	}
	
	// Perform complex logic...
}
```

### Avoiding String Allocations (CPU)

Constructing new strings using `+` or string interpolation (`$""`) every frame creates memory garbage, forcing the Garbage Collector to run and cause micro-stutters. If updating UI, only update the text when the value actually changes.

```csharp
private int _lastScore = -1;

protected override void OnUpdate()
{
	if ( GameManager.Score != _lastScore )
	{
		// Only allocate a new string when the score actually changes
		_lastScore = GameManager.Score;
		ScoreLabel.Text = $"Score: {_lastScore}";
	}
}
```

### Adjusting Shadow Casting (GPU)

Shadows are very expensive. For every light that casts shadows, the engine has to render the scene again from the light's perspective. 

1. Ensure only your primary light source (like the Sun) casts high-quality shadows. 
2. Disable the `Shadows` property on `ModelRenderer` or `SpriteRenderer` for small debris, grass, or distant objects that don't need accurate shadows.

## Configuration

When diagnosing performance issues, check these common heavy configurations:

| Component Property | Impact | How to Optimize |
|---|---|---|
| `ModelRenderer.Shadows` | GPU | Disable on small or unimportant objects. |
| `PointLight.Shadows` | GPU/CPU | Avoid overlapping multiple shadow-casting point lights. |
| `CameraComponent.ZFar` | GPU | Reduce the far clip plane distance to avoid rendering distant geometry. |
| `Terrain.HeightmapResolution` | VRAM/GPU | Use a smaller heightmap scale rather than massive 8192x8192 maps. |

## Troubleshooting

:::danger "My game randomly stutters every few seconds!"
This is almost certainly a Garbage Collection (GC) spike. Use `profiler` in the console to capture a deep trace, open it in Firefox Profiler, and look for GC Events. Usually, this means you are using `new` to create objects, arrays, or strings inside `OnUpdate`. Switch to object pooling or caching.
:::

:::warning "Frame rate drops when looking at a specific object"
If looking at an object tanks your FPS, it is likely GPU-bound. Check if the object has a highly complex custom shader, excessive polygon count, or is casting complex volumetric shadows.
:::

## Related Pages
- [Profiling & Overlays](profiling.md)
- [Component Queries](../../how-to/component-queries.md)
