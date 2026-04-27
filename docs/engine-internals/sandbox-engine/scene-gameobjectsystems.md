---
title: "Sandbox.Engine: GameObjectSystems"
icon: "🔧"
sources:
  - engine/Sandbox.Engine/Scene/GameObjectSystems/
created: 2026-04-27
updated: 2026-04-27
---

# GameObjectSystems

The `GameObjectSystems` directory contains specialized classes that inherit from `GameObjectSystem`. These are engine-level managers that listen to specific stages of the scene lifecycle to process batches of objects or manage global state without requiring individual components on every GameObject to tick themselves.

## Core Systems Implemented

Several critical engine features are implemented as `GameObjectSystem`s:

1.  **`ScenePhysicsSystem`:** Ticks the underlying `PhysicsWorld` during the `PhysicsStep` stage. It manages the registration of rigidbodies and colliders, and propagates physics events (like `OnIntersectionStart`) back to the managed C# components.
2.  **`ClutterGridSystem`:** Manages the generation and streaming of infinite clutter (grass, rocks). It handles background generation jobs and only renders clutter tiles near the active camera.
3.  **`SceneAnimationSystem`:** Batches and processes skeletal animation updates.
4.  **`NavMeshAgentSystem`:** Coordinates the movement and pathfinding updates for all AI navigation agents.
5.  **`SceneSoundscapeSystem`:** Manages ambient soundscapes based on the player's location.
6.  **`SceneSpriteSystem`:** Handles the batching and rendering of 2D sprites within the 3D scene.

## Common Engine Patterns

If you are contributing to the engine and need to add a new global behavior, you should often use a `GameObjectSystem`.

```csharp
// Example pattern for an engine system
[Expose]
sealed class MyCustomSystem : GameObjectSystem<MyCustomSystem>
{
    public MyCustomSystem( Scene scene ) : base( scene )
    {
        // Bind to a specific stage in the tick pipeline
        Listen( Stage.FinishUpdate, 0, ProcessData, "MySystem.Process" );
    }

    void ProcessData()
    {
        // Centralized logic executed once per frame
    }
}
```

## Performance & Lifecycle Notes

*   **Execution Order:** The integer passed to `Listen( stage, order, action, name )` dictates the execution order within that specific stage. Engine developers must carefully coordinate these orders so that, for example, animations are updated before rendering submissions occur.
*   **Scene Scope:** A `GameObjectSystem` is tied to a specific `Scene` instance. If multiple scenes are loaded (e.g., the main game and a UI background scene), each scene will have its own independent instances of these systems.

## Related Pages
- [Scene Architecture](../../systems/scene-system/architecture.md)
- [Sandbox.Engine.Scene](scene.md)
