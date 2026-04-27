---
title: "Audio"
icon: "🔊"
sources:
  - engine/Sandbox.Engine/Systems/Audio/
updated: 2026-04-25
created: 2026-04-27
---

# Audio

In s&box, you don't usually play raw audio files. You play **Sound Events** — small assets (`.sound`) that wrap one or more audio files with playback rules: pitch randomization, attenuation curves, max distance, mixer routing. The same gunshot sound played by a Sound Event sounds different at different ranges; the engine handles all of that.

For continuous audio attached to a moving thing (engine hum, ambient loop), you'll use a `SoundPointComponent` or `SoundBoxComponent` instead. For one-shots that just need to happen, the static `Sound.Play()` is what you want.

```csharp
using Sandbox;

public sealed class GunshotPlayer : Component
{
    [Property] public SoundEvent GunshotSound { get; set; }

    public void Fire()
    {
        Sound.Play( GunshotSound, WorldPosition );
    }
}
```

That's it for fire-and-forget. The engine handles spatial mixing, falloff, and disposal.

## What you'll actually use

The shortest list:

- **`Sound.Play( soundEvent, worldPosition )`** — fire and forget. Most gunshots, footsteps, impact sounds, UI clicks.
- **`SoundPointComponent`** — a Sound Event attached to a GameObject. Loops, follows the object, can be controlled (volume/pitch) at runtime. Use this for engine sounds, machines, ambient sources.
- **`SoundBoxComponent`** — like `SoundPointComponent` but plays from anywhere inside an axis-aligned volume rather than a single point. Ambient room tones.
- **`AudioListener`** — overrides where the local client hears audio from. Default is the camera; you might move it to a player's head for first-person.
- **`SoundscapeTrigger`** — switches the active soundscape (background ambience layer) when the listener walks into a volume.
- **Mixers** — group sounds into buses (Music / SFX / Voice / UI) so you can wire them to global volume sliders.
- **`DSPVolume`** — applies real-time DSP effects (reverb, low-pass filter) when the listener is inside a volume. Used for caves, water, indoor reverb.

For voice chat, `Voice` is a separate component that captures microphone input and replicates it through the networking system. For lip sync from playing audio, `LipSync` drives morph weights on a model from the spectrum of the sound it's playing.

## Music and streaming audio

For programmatic music streaming with playback control (volume, position, spectrum visualization), don't use Sound Events — use `MusicPlayer`. It lives in [Media → Audio (Streaming)](../media/audio.md). Sound Events are for one-shot sound effects and looped point/box sources; `MusicPlayer` is for long-form streamed audio with API control.

## Patterns

**Manual volume curve on a continuous source.** If you have a `SoundPointComponent` and want its volume to track a gameplay variable (e.g., engine pitch tracks car speed):

```csharp
public class EngineSound : Component
{
    [RequireComponent] public SoundPointComponent Sound { get; set; }
    [Property] public Rigidbody Body { get; set; }

    protected override void OnUpdate()
    {
        if ( Body is null ) return;
        var speed = Body.Velocity.Length;
        Sound.Volume = MathX.Remap( speed, 0, 600, 0.2f, 1.0f );
        Sound.Pitch  = MathX.Remap( speed, 0, 600, 0.8f, 1.4f );
    }
}
```

**Settings menu volume sliders.** Route sounds through mixers in their Sound Event configuration, then adjust mixer volume globally:

```csharp
Mixer.FindMixerByName( "SFX" ).Volume = 0.5f;
```

This affects every Sound Event routed through that mixer in the entire scene.

## Things that catch people out

**Sound doesn't play.** Almost always one of: the `SoundEvent` property isn't assigned in the Inspector, the listener (camera) is too far away from the source, or the GameObject playing it is disabled.

**Sound plays at full volume regardless of distance.** The Sound Event is configured as 2D (UI sound) instead of 3D (positional). Check the `.sound` asset's settings.

**First-play hitch on a long sound.** Toggle the Sound Event's Precache flag — the file gets loaded at map load instead of when the sound first plays.

**Stopping a `Sound.Play()` sound mid-playback.** `Sound.Play()` returns a `SoundHandle` — keep the handle if you need to stop, fade, or move the sound later:

```csharp
var handle = Sound.Play( GunshotSound, WorldPosition );
handle.Volume = 0.5f;
handle.Stop( fadeTime: 0.25f );
```

`SoundHandle` also exposes `Position`, `Pitch`, `Paused`, and `IsValid()` — useful if you want a fire-and-then-control pattern without going to `SoundPointComponent`. For continuous looping audio attached to a moving GameObject, `SoundPointComponent` is still the better choice.

**Audio in a dedicated server.** Dedicated servers don't process audio at all. `Sound.Play` calls are no-ops there. Don't rely on audio side-effects in server-authoritative logic.

## Read next

- [Audio Components](sound-components.md) — `SoundPointComponent`, `SoundBoxComponent`, `SoundscapeTrigger`, `AudioListener` API
- [Mixers](mixer.md) — bus routing and ducking
- [DSP Volume](dsp-volume.md) — applying reverb / filtering by area
- [Media → Audio (Streaming)](../media/audio.md) — `MusicPlayer` for programmatic music
- [Play Sounds from Code](../../how-to/play-sounds.md) — quick recipe
