---
title: "Create a NavMesh Obstacle"
icon: "🚧"
sources:
  - engine/Sandbox.Engine/Scene/Components/Navigation/NavMeshArea.cs
created: 2026-04-27
updated: 2026-04-27
---

# Create a NavMesh Obstacle

Learn how to dynamically block off an area of the NavMesh at runtime, forcing AI agents to find a new path around the obstacle.

## Quick Working Example

Add this script to an object (like a door or a fallen tree) that has a `NavMeshArea` component. The `NavMeshArea` component inherits from `VolumeComponent`, meaning its physical shape is defined by its `SceneVolume` property.

```csharp
using Sandbox;

public sealed class ObstacleController : Component
{
    [RequireComponent] public NavMeshArea Area { get; set; }

    public void BlockPath()
    {
        // Tells the NavMesh to treat this volume as a solid obstacle
        Area.IsBlocker = true;
    }

    public void OpenPath()
    {
        // Tells the NavMesh agents can walk through here again
        Area.IsBlocker = false;
    }
}
```

### A Closing Door

A common pattern is having a physical door that opens and closes. When it closes, you want agents to realize they can't walk through it anymore.

1. Create an Empty GameObject in the doorway.
2. Attach a `NavMeshArea` to it.
3. In the Inspector, adjust the `SceneVolume` property to tightly fit the size of the closed doorway.
4. When your door animation or physics finishes closing, call `Area.IsBlocker = true`.

```csharp
protected override void OnUpdate()
{
    if ( Input.Pressed( "Use" ) )
    {
        IsOpen = !IsOpen;
        
        // ... play door animation ...

        // Update the NavMesh instantly
        if ( Area != null )
        {
            Area.IsBlocker = !IsOpen;
        }
    }
}
```

## Configuration

| Property | Description |
|:---|:---|
| `IsBlocker` | When `true`, NavMesh generation in this area will be completely disabled, creating a hole in the navigation mesh. |
| `Area` | The specific `NavMeshAreaDefinition` to apply. Useful if you don't want to completely block the path, but instead want to mark it as a hazard (like a swamp) that agents should try to avoid. |
| `SceneVolume` | Defines the 3D shape and size of the area (e.g. Box, Sphere, Capsule). |

## Limitations & Performance

While updating a `NavMeshArea` is fast, it's not completely free. The engine has to recalculate the NavMesh locally around the volume.
- You can safely have dozens of obstacles toggling on and off.
- Avoid attaching a blocking `NavMeshArea` to an object that is constantly moving every frame (like a physics ball rolling down a hill), as it will force the NavMesh to rebuild every single frame and tank performance.

## Related Pages
- [NavMesh Areas](../systems/navigation/navmesh-areas/index.md)
- [NavMesh Agent](../systems/navigation/navmesh-agent.md)
