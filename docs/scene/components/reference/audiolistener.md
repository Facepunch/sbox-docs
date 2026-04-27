---
title: "Audio Listener"
icon: "👂"
sources:
  - engine/Sandbox.Engine/Scene/Components/Audio/AudioListener.cs
updated: 2026-04-25
created: 2026-04-27
---

# Audio Listener

> **New to audio?** This is the reference for the `AudioListener` component. The audio system as a whole is documented at **[Audio](../../../systems/audio/index.md)**.

By default the local client hears audio from the active camera's position and rotation. An `AudioListener` overrides that — while one is enabled in the scene, audio is heard from the listener's `GameObject` instead.

This is useful when the camera is somewhere your character "isn't" — third-person shoulder cams, security cameras, cinematic shots — and you still want sound to be relative to the player's head.

## Quick Working Example

```csharp
public class HeadListener : Component
{
    protected override void OnStart()
    {
        // Attach an audio listener to the player's head bone
        AddComponent<AudioListener>();
    }
}
```

## Common Patterns

### Third-person stereo from the head, panning from the camera

Default behavior. Place the listener on the player's head and leave `UseCameraDirection = true`. Sounds are spatialised relative to the head's position but the stereo field follows the camera, which feels right because the camera is what the player is steering.

### Fully head-locked listener

For first-person VR-style audio where the head's rotation is what should drive panning, set `UseCameraDirection = false`.

```csharp
listener.UseCameraDirection = false;
```

### Switching listeners

Disabling one `AudioListener` and enabling another is the supported way to "swap" the listening point — the engine adds and removes listeners on enable/disable, and the active set is queried by the audio system.

## Properties

| Property | Default | Description |
|---|---|---|
| `UseCameraDirection` | `true` | If true, the listener's rotation is taken from `Scene.Camera.WorldRotation` instead of this GameObject's rotation. Position is always taken from this GameObject. |

:::warning "No camera in scene"
If `UseCameraDirection` is true but `Scene.Camera` is invalid, the listener falls back to the GameObject's own rotation for that frame. This means the listener never errors out, but switching between cameras during a scene transition can cause a one-frame rotation pop.
:::

:::tip "Multiple listeners"
The component itself does not prevent multiple listeners — each enabled `AudioListener` registers its own listener handle. The audio system decides what to do; in practice you should keep at most one enabled.
:::

## Related Pages

- [Sound Point](soundpointcomponent.md)
- [Voice](voice.md)
- [Camera Component](cameracomponent.md)
