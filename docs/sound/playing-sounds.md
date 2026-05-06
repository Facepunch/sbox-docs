---
title: "Playing Sounds"
icon: "🔊"
created: 2026-05-04
updated: 2026-05-04
---

# Playing Sounds

s&box provides several ways to play sounds in code. You can fire-and-forget a sound at a world position, control it via a `SoundHandle`, attach it to a `GameObject`, or use scene components entirely.

## SoundEvent

Most sounds in s&box are defined as **SoundEvent** assets (`.sound` files). A SoundEvent bundles one or more audio files together with settings like volume range, pitch range, 3D distance, occlusion, and a selection mode (random, sequential, etc.).

Reference a SoundEvent from a property on your component:

```csharp
[Property] public SoundEvent FootstepSound { get; set; }
```

You can also load one by path at runtime:

```csharp
var sound = ResourceLibrary.Get<SoundEvent>( "sounds/footsteps/concrete.sound" );
```

:::tip
The **UI** flag on a SoundEvent makes it play as a flat 2D sound with no distance attenuation - useful for interface and HUD audio.
:::

## Sound.Play

`Sound.Play` is the primary static API for playing sounds in code. It returns a `SoundHandle` you can use to control the sound later.

### Basic playback

```csharp
// By SoundEvent reference
Sound.Play( FootstepSound );

// By asset path (string)
Sound.Play( "sounds/explosion.sound" );

// By asset name (string), can be placed in any folder but must be unique
Sound.Play( "explosion_large" );

```

Both overloads are identical in behaviour - the string version simply looks up the SoundEvent by name.

### 3D positional sound

Pass a world-space `Vector3` to place the sound in the scene:

```csharp
Sound.Play( ExplosionSound, WorldPosition );
Sound.Play( "sounds/explosion.sound", transform.Position );
```

Without a position the sound defaults to the world origin `(0, 0, 0)`. For a sound that should follow an object, use `GameObject.PlaySound` instead (see below).

### Fade in

An optional `fadeInTime` parameter fades the volume up from silence over the given number of seconds:

```csharp
Sound.Play( AmbientSound, WorldPosition, fadeInTime: 2.0f );
```

## SoundHandle

Every `Sound.Play` call returns a `SoundHandle`. Keep a reference to it when you need to change or stop the sound after it starts.

```csharp
SoundHandle _engineLoop;

void StartEngine()
{
    _engineLoop = Sound.Play( EngineLoopSound, WorldPosition );
    _engineLoop.Volume = 0.8f;
    _engineLoop.Pitch = 1.2f;
}
```

### Key properties

| Property | Description |
|---|---|
| `Position` | World-space position of the sound |
| `Volume` | Volume multiplier (default `1.0`) |
| `Pitch` | Pitch multiplier (default `1.0`, `2.0` = one octave up) |
| `Distance` | Max audible distance in units (default 15 000) |
| `SpacialBlend` | `0` = fully 2D, `1` = fully 3D (default `1.0`) |
| `Occlusion` | Whether geometry can block the sound (default `true`) |
| `IsPlaying` | `true` while the sound is still active |
| `Paused` | Pause or resume without stopping |
| `Time` | Playback position in seconds (seekable) |
| `TargetMixer` | Route to a specific audio mixer |

### Stopping a sound

```csharp
// Instant stop
_engineLoop.Stop();

// Fade out over 1.5 seconds
_engineLoop.Stop( 1.5f );
```

After `Stop()` the handle becomes invalid. Accessing properties on an invalid handle is safe - they simply do nothing.

### Stop everything

```csharp
Sound.StopAll( fade: 0.5f );
```

## GameObject.PlaySound

`GameObject.PlaySound` plays a sound that **follows** the GameObject's world position every frame. This is the recommended way to play sounds on moving objects - footsteps, projectile impacts, character voices.

```csharp
// Plays the sound at this object's position and follows it
var handle = GameObject.PlaySound( FootstepSound );

// Plays the sound at around the player's head - offset is local
var handle = GameObject.PlaySound( FootstepSound, Vector3.Up * 64f );
```

The returned `SoundHandle` is just like any other - you can adjust volume, pitch, or call `Stop()` on it.

### Stopping all sounds on a GameObject

```csharp
// Stops every sound currently following this GameObject
GameObject.StopAllSounds();

// Fade out over 0.5 seconds
GameObject.StopAllSounds( fadeOutTime: 0.5f );
```

:::info
`GameObject.PlaySound` internally sets `SoundHandle.Parent` and `SoundHandle.FollowParent = true`. You can do this manually for sounds started with `Sound.Play` if you need the same behaviour:

```csharp
var handle = Sound.Play( FootstepSound, WorldPosition );
handle.Parent = GameObject;
handle.FollowParent = true;
```
:::

## Audio components

For sounds that don't need runtime code control, add one of these components to a `GameObject` in the editor:

| Component | Description |
|---|---|
| `SoundPointComponent` | Plays a sound at a fixed world point. Supports auto-play, looping, and a randomised repeat interval. |
| `SoundBoxComponent` | Like `SoundPointComponent`, but the source position is constrained to a box region. |
| `SoundscapeTrigger` | Blends in an ambient soundscape when the audio listener enters the trigger. |
| `AudioListener` | Overrides the listening position. Without this, audio is heard from the camera. |

### AudioListener

By default the player hears sounds from the camera's position. Add an `AudioListener` component to a different `GameObject` (e.g., the player's head bone) to move the listening point.
