---
title: Sandbox.Engine.Scene
icon: "🌲"
sources:
  - engine/Sandbox.Engine/Scene/
updated: 2026-04-25
created: 2026-04-27
---

# Sandbox.Engine.Scene

The `Sandbox.Engine.Scene` directory contains the core implementation of the Scene System, the primary way developers construct and manage game worlds in s&box. This includes the internal definitions of `GameObject`s, `Component`s, and the lifecycle management of a Scene.

## Quick Start Example

As an engine contributor, you might interact with the internal Scene representation to process GameObjects directly:

```csharp
using Sandbox;

internal class SceneAnalyzer
{
    public void AnalyzeActiveScene()
    {
        // The active Scene is exposed as Game.ActiveScene.
        // See engine/Sandbox.Engine/Game/Game/Game.Scene.cs.
        var currentScene = Game.ActiveScene;
        if ( currentScene == null ) return;

        // Walk every GameObject in the scene graph.
        // GetAllObjects is defined on GameObject (Scene derives from GameObject).
        foreach ( var go in currentScene.GetAllObjects( enabled: false ) )
        {
            Log.Info( $"Object: {go.Name} has {go.Components.Count} components." );
        }
    }
}
```

## Extending and Utilizing

As a game developer, you primarily interact with these classes indirectly by inheriting from `Component` in your own scripts. However, engine developers might need to understand the deeper workings:

- **Lifecycle:** Understanding when `OnAwake`, `OnStart`, `OnUpdate`, and `OnDestroy` are called internally by the Scene manager.
- **Serialization:** How GameObjects and Components are serialized to and from Prefab files (`.prefab`) or Scene files (`.scene`).
- **Networking:** The mechanisms by which `[Sync]` properties on Components are tracked and replicated by the networking system.

- **Hierarchy Depth:** Deeply nested GameObject hierarchies can impact performance, as transform updates must propagate down the tree.
- **Component Ticking:** Overuse of `OnUpdate` across thousands of components can become a bottleneck. Consider system-based updates or disabling components when not needed.

## Serialization and Prefabs

The `GameObject` class contains extensive logic for saving and loading its state via JSON, enabling scene saving, prefabs, and networking:

- **`GameObject.Serialize.cs`**: Handles converting a GameObject (and all its children and components) into a `JsonObject`. This is crucial for saving `.scene` files, transferring network state, and cloning objects. The process is governed by `SerializeOptions` to determine what data to include (e.g., `SceneForNetwork` to exclude non-networked fields). The system utilizes an internal `DiffObjectDefinitions` tracker to determine minimal json diffs.
- **`GameObject.Prefab.cs`**: Manages the relationship between a live GameObject and its source `.prefab` file. Prefab instances are tracked using `PrefabInstanceData`. It includes concepts like `IsOutermostPrefabInstanceRoot` and handles updating references through methods like `UpdateFromPrefab()` to revert local overrides and match the source file again, or `BreakFromPrefab()` to sever the connection entirely.

## Related Pages
- [Scene System Architecture](../../systems/scene-system/architecture.md)
- [GameObjectSystems](scene-gameobjectsystems.md)
- [Scene Networking](scene-networking.md)
