---
title: "Prefabs"
icon: "📦"
sources:
  - engine/Sandbox.Engine/Scene/GameObject/GameObject.Clone.cs
created: 2026-04-27
updated: 2026-04-27
---

# Prefabs

A Prefab is a saved GameObject template that can be reused across multiple scenes or instantiated dynamically at runtime.

## Quick Working Example

```csharp
using Sandbox;

public sealed class SpawnerComponent : Component
{
    // Assign a Prefab in the Editor Inspector
    [Property] public GameObject MyPrefab { get; set; }

    protected override void OnUpdate()
    {
        if ( Input.Pressed( "Attack1" ) && MyPrefab != null )
        {
            // Create a new instance of the prefab in the world
            var spawnedObj = MyPrefab.Clone( GameObject.WorldTransform );
            Log.Info( $"Spawned: {spawnedObj.Name}" );
        }
    }
}
```

### Creating a Prefab
To create a Prefab, right-click any GameObject in the Scene hierarchy and select **Convert to Prefab**. This saves it as an asset on disk (e.g., `my_object.prefab`).

### Editing a Prefab
If you select a prefab instance in the scene, it appears blue, and you cannot edit its hierarchy directly. 

![They're blue](./images/they-re-blue.png)

To modify the master template:
1. Double-click the `.prefab` file in your Asset Browser to open it in isolation.
2. Make your changes and save. All instances across all scenes will update automatically.

### Unlinking a Prefab
If you want to turn a prefab instance back into regular GameObjects (so you can modify it freely without affecting the master template), right-click it in the scene and choose **Unlink from Prefab**. 
In code, you can do this by calling:
```csharp
myGameObject.BreakFromPrefab();
```

## Navigation Hub

- [**Instance Overrides**](instance-overrides.md) - Learn how to modify specific properties on a single prefab instance without changing the master template.
- [**Prefab Templates**](prefab-templates.md) - Use variables to make customizable prefabs.

## Troubleshooting

:::warning "I can't delete or move children of a prefab in the scene!"
By design, you cannot radically alter the hierarchy of a prefab instance while it is linked to the master template. To add or remove child objects, either double-click the prefab to edit the master, or right-click the instance and select **Unlink from Prefab**.
:::

:::danger "My prefab spawns at 0,0,0 instead of where I want it!"
If you call `MyPrefab.Clone()` without arguments, it spawns at the world origin. Use `MyPrefab.Clone( WorldTransform )` or manually set `spawnedObj.WorldPosition` immediately after cloning.
:::

## Related Guides
- [**Spawn a Prefab**](../../how-to/spawn-prefab.md)
