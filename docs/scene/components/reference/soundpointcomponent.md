---
title: "Sound Point"
icon: "🔊"
sources:
  - engine/Sandbox.Engine/Scene/Components/Audio/SoundPointComponent.cs
  - engine/Sandbox.Engine/Scene/Components/Audio/BaseSoundComponent.cs
updated: 2026-04-25
created: 2026-04-27
---

# Sound Point

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `SoundPointComponent` component.

The `SoundPointComponent` plays an audio event at a specific point in the 3D world, allowing for spatial audio and distance attenuation.

## Quick Working Example

```csharp
using Sandbox;

public sealed class AlarmTrigger : Component
{
    [RequireComponent] public SoundPointComponent SoundPoint { get; set; }

    protected override void OnStart()
    {
        // Start playing the sound event assigned in the inspector
        SoundPoint.StartSound();
    }

    public void TurnOffAlarm()
    {
        // Stop the sound
        SoundPoint.StopSound();
    }
}
```

### Looping Sounds

If you have an ambient sound (like a humming generator or a waterfall), you can set `PlayOnStart` and `Repeat` to true. If the `SoundEvent` itself is a looping sound, it will continue indefinitely. If it is a one-shot sound, the `Repeat` property will re-trigger it based on the `MinRepeatTime` and `MaxRepeatTime`.

### Modifying Sound Properties via Code

While most properties are set in the Editor, you can adjust the volume and pitch of the sound programmatically by enabling `SoundOverride`.

```csharp
public void RevEngine( float power )
{
    SoundPoint.SoundOverride = true;
    SoundPoint.Pitch = 1.0f + power; // Increase pitch based on power
    SoundPoint.Volume = 0.5f + (power * 0.5f);
}
```

## Configuration

The `SoundPointComponent` inherits many properties from `BaseSoundComponent`.

| Property | Description |
|:---|:---|
| `SoundEvent` | The audio resource to play. |
| `PlayOnStart` | If true, the sound starts playing as soon as the component is enabled. |
| `StopOnNew` | If true, any currently playing sound from this component stops before starting a new one. |
| `TargetMixer` | The audio mixer channel to route this sound through. |

### Sound Overrides

Enable `SoundOverride` to manually control the output.

| Property | Description |
|:---|:---|
| `Volume` | Scales the volume of the sound (0 to 1). |
| `Pitch` | Scales the pitch/speed of the sound (0 to 2). |
| `Force2d` | If true, the sound plays globally at full volume, ignoring its 3D position. |

### Repeat

Enable `Repeat` to automatically re-trigger the sound.

| Property | Description |
|:---|:---|
| `MinRepeatTime` / `MaxRepeatTime` | The sound will wait a random duration between these two values before playing again. |

### Distance Attenuation (Falloff)

Enable `DistanceAttenuationOverride` to manually control how far the sound travels, overriding the settings baked into the `SoundEvent` asset.

| Property | Description |
|:---|:---|
| `Distance` | The maximum distance the sound can be heard from. |
| `Falloff` | A curve defining how the volume drops off over the distance. |

## Troubleshooting

:::warning Sound Not Playing
Make sure you have assigned a valid `SoundEvent` in the inspector. A `SoundEvent` is required for the component to know what audio data to play.
:::

## Related Pages
- [Audio System](../../../systems/audio/index.md)
