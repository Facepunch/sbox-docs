---
title: "Mixers"
icon: "🎛️"
sources:
  - engine/Sandbox.Engine/Systems/Audio/Mixer.cs
  - engine/Sandbox.Engine/Systems/Audio/MixerHandle.cs
updated: 2026-04-25
created: 2026-04-27
---

# Mixers

> **New to audio?** Read **[Audio](index.md)** first — that's the overview. This is the reference for mixers; the audio landing covers the full picture.

Mixers are channels that group and process multiple sounds together, allowing you to independently control the volume, mute state, and spatialization of different categories of audio.

## Quick Working Example

```csharp
using Sandbox;
using Sandbox.Audio;

public sealed class MixerController : Component
{
	// Expose a MixerHandle property so you can select the mixer in the Editor.
	// We default it to the standard "Game" mixer.
	[Property] public MixerHandle TargetMixer { get; set; } = "Game";

	public void SetVolume( float volume )
	{
		// Get the actual Mixer instance from the handle
		var mixer = TargetMixer.Get();
		if ( mixer != null )
		{
			mixer.Volume = volume;
		}
	}

	public void ToggleMute()
	{
		var mixer = TargetMixer.Get();
		if ( mixer != null )
		{
			mixer.Mute = !mixer.Mute;
		}
	}
}
```

### Adjusting Global Audio Settings
A common scenario is building an options menu where players can adjust volume sliders for different categories.

```csharp
public void UpdateMusicVolume( float newVolume )
{
	// You can find a mixer by its name
	var musicMixer = Mixer.FindMixerByName( "Music" );
	if ( musicMixer != null )
	{
		musicMixer.Volume = newVolume;
	}
}
```

### Pausing Game Audio
When opening a pause menu, you might want to mute all in-game sound effects but leave the UI and Music audible.

```csharp
public void OnGamePaused()
{
	var gameMixer = Mixer.FindMixerByName( "Game" );
	if ( gameMixer != null )
	{
		gameMixer.Mute = true;
	}
}
```

### Applying Global Effects
You can apply digital signal processing (DSP) effects to a mixer, such as when using a `DspVolume` component in your scene, which targets a specific mixer channel to apply reverb.

## Configuration (Mixer)

When configuring a Mixer, these are the common properties available:

| Property | Description |
|:---|:---|
| `Volume` | The overall volume multiplier for all sounds passing through this mixer. |
| `Mute` | If true, all sounds passing through this mixer (and its children) will be silenced. |
| `Solo` | If true, only this mixer (and others with Solo enabled) will be heard. |
| `Spacializing` | A multiplier for how much 3D spatialization is applied. Useful to set to `0` for UI or Music mixers so they play purely in 2D. |
| `DistanceAttenuation`| A multiplier for how quickly sounds quiet down over distance. |
| `Occlusion` | A multiplier for how much level geometry muffles sounds on this mixer. |

## Troubleshooting

:::danger "Mixer Handle is null!"
If you use `Mixer.FindMixerByName()` or `MixerHandle.Get()` and it returns null, ensure that the mixer actually exists in your project settings. Names are case-insensitive, but they must exactly match an existing mixer channel.
:::

:::warning "Sounds aren't spatializing!"
If you route a 3D sound to the "UI" or "Music" mixer and it plays as a flat 2D sound, check the `Spacializing` property of that mixer. By default, UI and Music mixers often have their `Spacializing` set to `0`.
:::

## Related Pages
- [Audio System Overview](index.md)
- [DSP Volume](dsp-volume.md)
