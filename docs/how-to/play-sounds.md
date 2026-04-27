---
title: "Play Sounds from Code"
icon: "🎵"
sources:
  - engine/Sandbox.Engine/Systems/Audio/Sound.Play.cs
  - engine/Sandbox.Engine/Systems/Audio/SoundHandle.cs
updated: 2026-04-25
created: 2026-04-27
---

# Play Sounds from Code

> **New to audio?** This is the recipe for one specific task: playing a sound from code. The full audio system is at **[Audio](../systems/audio/index.md)**.

To spawn one-off sounds like footsteps, gunshots, or UI clicks directly from your C# code, use the static `Sound.Play()` API.

## Quick Working Example

```csharp
using Sandbox;

public sealed class WeaponFire : Component
{
    // Expose a property so you can select the sound in the Editor
    [Property] public SoundEvent FireSound { get; set; }

    public void Fire()
    {
        // Play the sound exactly where this GameObject is located
        var handle = Sound.Play( FireSound, WorldPosition );
        
        // Optionally tweak the sound after it starts playing
        if ( handle != null )
        {
            // Randomize the pitch slightly for variety
            handle.Pitch = Game.Random.Float( 0.9f, 1.1f );
        }
    }
}
```

## Variations

### Playing UI Sounds

If you want a sound to play at full volume directly in the player's ears regardless of where they are in the 3D world (like a menu click or a notification), omit the position parameter or use a string name.

```csharp
// Plays a UI sound without 3D spatialization.
Sound.Play( "ui.button.click" );
```

### Stopping Sounds Early

If you need to cut a looping sound off before it finishes (like an engine sound or an alarm), hold onto the `SoundHandle` it returns and call `Stop()` when needed.

```csharp
SoundHandle _engineLoop;

public void StartEngine()
{
    // Start the sound and save the handle
    _engineLoop = Sound.Play( "car.engine.loop", WorldPosition );
}

public void StopEngine()
{
    // Stop the sound over a 0.5 second fade out
    _engineLoop?.Stop( 0.5f );
    _engineLoop = null;
}
```

### Modifying Playing Sounds

You can continuously update a playing sound's properties. For example, changing the pitch of an engine based on speed.

```csharp
protected override void OnUpdate()
{
    if ( _engineLoop != null && _engineLoop.IsPlaying )
    {
        // Update position so the sound follows the car
        _engineLoop.Position = WorldPosition;
        
        // Change pitch based on speed
        _engineLoop.Pitch = 1.0f + (Speed / MaxSpeed);
    }
}
```

### Stopping All Sounds

During dramatic moments or when transitioning between major game states, you might want to stop everything playing in the scene.

```csharp
// Stop all currently playing sounds with a 1-second fade out
Sound.StopAll( 1.0f );
```

## Configuration (SoundHandle)

When you call `Sound.Play`, it returns a `SoundHandle` object. Here are the common properties you can modify while the sound is playing:

| Property | Description |
|:---|:---|
| `Volume` | The current volume (multiplier) of the sound. |
| `Pitch` | The pitch multiplier (1.0 is normal, 2.0 is double speed/pitch). |
| `Position` | The 3D position in the world where the sound is emanating from. |
| `IsPlaying` | Returns true if the sound has not finished yet. |
| `TargetMixer` | The mixer channel this sound routes into. |
| `ListenLocal` | If true, forces the sound to be 2D (like UI), bypassing spatialization. |

## Troubleshooting

:::warning Sound Handle is Null
`Sound.Play()` will return `null` if you run it in a headless server environment (dedicated servers don't play audio), or if it couldn't find the requested `SoundEvent`. Always use the null conditional operator (`?.`) when accessing a returned handle.
:::

## Related Pages
- [Audio System Overview](../systems/audio/index.md)
