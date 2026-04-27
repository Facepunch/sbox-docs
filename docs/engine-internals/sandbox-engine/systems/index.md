---
title: "Sandbox.Engine: Systems"
icon: "🌐"
sources:
  - engine/Sandbox.Engine/Systems/Render/
  - engine/Sandbox.Engine/Systems/Audio/
  - engine/Sandbox.Engine/Systems/Physics/
updated: 2026-04-25
created: 2026-04-27
---

# Systems

The `Systems` directory within `Sandbox.Engine` houses the managed implementations of major engine subsystems. Unlike `GameObjectSystem`s, which exist specifically to operate on Scene entities, these systems provide lower-level, global functionality that spans the entire engine context.

## Core Engine Systems

### SceneSystem (`Systems/SceneSystem/`)
The foundational managed representation of the game world.
*   **GameObjects & Components:** Handles the lifecycle, transform hierarchy, and updating of all active GameObjects and their Components.
*   **Scene Execution:** Manages the main game loop ticks (`Update`, `FixedUpdate`, `PreRender`).

### [Networking](networking.md) (`Systems/Networking/`)
Handles the replication of state across the client-server boundary.
*   **`SceneNetworkSystem`:** Orchestrates the serialization and synchronization of `GameObject` structures and `NetworkComponent` state between hosts and clients.

### UI (`Systems/UI/`)
The underlying implementation of the engine's Razor-based UI framework (distinct from `Sandbox.UI` public wrapper).
*   **`Panel` & Rendering:** Implements the core layout calculations, styles parser (CSS/SCSS), flexbox logic (via Yoga), and rasterization of panels to the rendering pipeline.
*   **Input Routing:** Dispatches mouse and keyboard events to the focused UI elements.

### [Movies](movies.md) (`Systems/Movies/`)
The managed API for playing and interacting with video content.
*   **`MoviePlayer`:** Wraps the native video playback engine to allow rendering video to textures or UI panels, managing audio sync, and play states.

### Render (`Systems/Render/`)

This subsystem acts as the primary managed interface to the Source 2 rendering pipeline.
*   **`Graphics` / `Graphics.Draw`:** Provides immediate-mode rendering capabilities (drawing shapes, meshes, and text) often used for debugging, custom tool rendering, or Editor UI.
*   **`CameraRenderer`:** The pipeline that takes a scene viewport, culls objects, and submits draw calls to the GPU.
*   **Shaders & Compute:** Contains wrappers for dispatching `ComputeShader`s and managing `GpuBuffer`s for advanced rendering techniques and async GPU readbacks.

### [Audio](audio.md) (`Systems/Audio/`)

The audio subsystem handles sound mixing, routing, and playback.
*   **`Mixer`:** Defines the hierarchical audio routing structure (e.g., routing sound effects to a "Master" bus). It manages effects processing (`Processors/`) and volume controls.
*   **`SoundHandle`:** Represents an actively playing sound instance. It handles 3D positioning (following a Transform), lip-sync generation, and pitch/volume manipulation over time.
*   **SteamAudio Integration:** Contains hooks for advanced spatialization and occlusion calculations provided by the Steam Audio library.

### Physics (`Systems/Physics/`)

This subsystem provides the managed API for the Rubikon physics engine.
*   **`PhysicsWorld`:** The global container for physics simulation. It handles raycasts/traces (`PhysicsTraceBuilder`), broad-phase queries, and simulation steps.
*   **`PhysicsBody` & `PhysicsShape`:** The core representations of rigidbodies and their collision hulls.
*   **`PhysicsJoint`:** Implementations of mechanical connections between bodies (hinges, springs, sliders).

### [Filesystem](filesystem.md) (`Systems/Filesystem/`)
Managed abstraction over the native mounting and virtual file I/O layer.
*   Provides structured access to reads/writes across differing mounted locations and package environments.

### Input (`Systems/Input/`)
Manages the global state of user input.
*   Tracks hardware states, action bindings, and relays generic input to subsystems (like UI or Scene).

### [Hotload](hotload.md) (`Systems/Hotload/`)
Implementation logic for the managed hot-reloading mechanism that lets developers alter code without restarting the engine.

*   **Garbage Collection:** Because these systems sit at the very bottom of the engine's call stack and are executed thousands of times per frame (especially in the case of `Graphics.Draw` or `PhysicsTrace`), they are designed to be zero-allocation on the managed heap. Structs are heavily favored over classes, and memory is often pinned or allocated natively.
*   **Threading:** The `Render` system submissions must generally happen on the main thread, while the `Audio` system often operates on a dedicated `MixingThread` to prevent audio dropouts during heavy visual frame spikes.

## Related Pages
- [Engine Launcher](../../launcher.md)
- [Foundational Systems Architecture](../../../systems/foundations/architecture.md)
