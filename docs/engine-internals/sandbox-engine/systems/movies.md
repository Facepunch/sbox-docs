---
title: "Sandbox.Engine: Movies"
icon: "📺"
sources:
  - engine/Sandbox.Engine/Systems/Movies/MoviePlayer.cs
created: 2026-04-27
updated: 2026-04-27
---

# Movies

The `Movies` subsystem is responsible for providing the managed API necessary to play video files and sequence animations via the Movie Maker timeline.

## Core Concepts

The system primarily revolves around the `MoviePlayer` component. Unlike raw video texture playback, this subsystem handles the orchestration of *Timeline* clips (`IMovieClip`), which can animate properties, trigger events, and control models over time.

### The `MoviePlayer` Component

`MoviePlayer` acts as the engine's timeline director within a `Scene`. It holds a reference to an `IMovieClip` and advances its playback time (`Position`) during the `OnUpdate` phase. Engine contributors extending the editor or writing native timeline extensions will interact with this component.

### Scene and Batch Scopes

When the `MoviePlayer` updates its position and applies track bindings, it pushes the `Scene` scope and initiates a `CallbackBatch`. This is critical for internal consistency:

```csharp
// Internal implementation excerpt from MoviePlayer.cs
internal IDisposable BeginApplyFrameInternal()
{
    var sceneScope = Scene.Push();

    // Batches property changes to ensure OnEnabled hooks 
    // are called in the correct order when toggling GameObjects.
    var batchScope = CallbackBatch.Batch();

    return new DisposeAction( () =>
    {
        batchScope?.Dispose();
        sceneScope?.Dispose();
    } );
}
```

### Track Binders and Targets

A `MovieResource` defines abstract tracks (e.g., "Move this object", "Animate this bone"). It does not contain the actual scene objects. 

When a `MoviePlayer` plays a clip, it uses a `TrackBinder` to map those abstract tracks to concrete `GameObject`s and `Component`s in the active `Scene`.

*   **`CreateTargets`**: If `CreateTargets` is enabled on the `MoviePlayer`, the engine will automatically instantiate new `GameObject`s to fulfill the track bindings if they do not already exist in the scene hierarchy.
*   **Applying Frames**: During `OnUpdate`, the `MoviePlayer` invokes `clip.Update( _position, Binder )`. This causes the binder to evaluate the keyframes for the current time and forcefully inject the values into the target `ITrackTarget` properties.

## Animation Playback Rates

A significant responsibility of the `MoviePlayer` is coordinating rigidbodies and model animations during playback. When a movie clip takes control of a `SkinnedModelRenderer`:

1.  It searches for attached `Rigidbody` components and sets `MotionEnabled = false` to prevent physics from fighting the timeline animation.
2.  It scales the `PlaybackRate` of the underlying `SceneModel` by the `MoviePlayer`'s `TimeScale`.
3.  When playback stops (or the object is no longer bound), it restores `MotionEnabled = true`.

## Troubleshooting

:::warning Deserialization Halts
The `MoviePlayer` will intentionally skip updating its position and track targets if it detects that its parent `GameObject` or the component itself is currently undergoing deserialization (`ComponentFlags.Deserializing`). This prevents half-initialized state from being permanently written into scene properties.
:::