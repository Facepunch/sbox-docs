---
title: "Obstacles"
icon: "🚫"
created: 2025-10-22
updated: 2026-04-12
sources:
  - engine/Sandbox.Engine/Scene/Components/Navigation/NavMeshArea.cs
---

# Obstacles

Obstacles completely remove a section of the NavMesh, preventing any agents from generating paths through that area. 

## Quick Working Example

You can toggle an obstacle at runtime. When you change the `IsBlocker` property, the engine will automatically update the NavMesh in the background.

```csharp
public class Forcefield : Component
{
	[RequireComponent] public NavMeshArea NavArea { get; set; }

	public void ActivateForcefield()
	{
		// Block agents from walking through this area
		NavArea.IsBlocker = true;
	}

	public void DeactivateForcefield()
	{
		// Allow agents to walk through this area again
		NavArea.IsBlocker = false;
	}
}
```

## In the Editor

1. Create a `GameObject` representing your obstacle.
2. Add a `NavMeshArea` component.
3. Configure the `SceneVolume` property on the component to match the size and shape of your obstacle (e.g., Box, Sphere, Capsule).
4. Ensure the **Is Blocker** property is checked.

When you view the NavMesh in the editor, you will see a hole cut out where your obstacle is placed.

![NavMesh obstacle blocking demo](./images/obstacle-example.mp4)

## Troubleshooting

:::warning Obstacle not blocking?
Ensure the `SceneVolume` on the `NavMeshArea` is actually large enough to intersect the floor. If the volume is floating above the NavMesh, it won't cut a hole in it.
:::

## Related Pages
- [NavMesh Areas Overview](index.md)
