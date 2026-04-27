---
title: "Sandbox.Engine: Scene Networking"
icon: "📡"
sources:
  - engine/Sandbox.Engine/Scene/Networking/
created: 2026-04-27
updated: 2026-04-27
---

# Scene Networking

The `Networking` directory within `Sandbox.Engine/Scene/` contains the internal implementation details of how the s&box Scene System synchronizes GameObjects, Components, and variables across network connections for multiplayer games.

## Internal Mechanisms

### Delta Snapshots

Instead of sending the entire state of the scene every tick, `SceneNetworkSystem` uses `DeltaSnapshotSystem`. It takes a snapshot of the `NetworkTable` for all networked objects. On subsequent ticks, it compares the current state against the previous snapshot and only transmits the differences (deltas).

### Ownership & Proxy State

A critical part of `NetworkObject` is determining who "owns" an object.
*   **Authority:** Typically the Host. The authority dictates the absolute truth of the object's state.
*   **Owner:** The client connection assigned to control the object.
*   **Proxy:** If a client does not own an object, it views that object as a "Proxy".

The `NetworkObject` class handles the complex state transitions during `NetworkSpawn`, temporarily assigning ownership to the Host during initialization before handing it off to the requested client, ensuring `OnAwake` logic executes securely.

## Common Engine Patterns

If you are modifying the engine's networking behavior:

1.  **Adding new Message Types:** If the Scene System needs a new way to sync a specific engine feature, you define a struct implementing `INetworkMessage` and register a handler in `SceneNetworkSystem`'s constructor.
    ```csharp
    // Inside SceneNetworkSystem constructor
    AddHandler<MyCustomMsg>( OnMyCustomMsgReceived );
    ```
2.  **Network Tables:** When a user marks a C# property with `[Sync]`, the Code Generator writes IL that intercepts the property setter to update the internal `NetworkTable`. Engine developers must ensure that `NetworkTable` remains performant, as it is queried constantly during delta generation.

## Related Pages
- [Client Architecture](../../systems/networking-multiplayer/client-architecture.md)
- [Server Architecture](../../systems/networking-multiplayer/server-architecture.md)
