---
title: "DSP Volume"
icon: "📊"
sources:
  - engine/Sandbox.Engine/Scene/Components/Audio/Dsp/DspVolume.cs
updated: 2026-04-25
created: 2026-04-27
---

# DSP Volume

> **New to audio?** Read **[Audio](index.md)** first — that's the overview. This is the reference for the `DspVolume` component; the audio landing covers the full picture.

The `DspVolume` component applies Digital Signal Processing (DSP) effects to a target audio mixer when the active listener is inside its bounds.

## Quick Working Example

```csharp
public class ReverbRoom : Component
{
	protected override void OnStart()
	{
		// Create a DspVolume
		var dspVolume = Components.GetOrCreate<DspVolume>();
		
		// Set the shape to a Box volume
		dspVolume.SceneVolume = new Sandbox.Volumes.SceneVolume()
		{
			Type = Sandbox.Volumes.SceneVolume.VolumeTypes.Box,
			Box = BBox.FromPositionAndSize( 0, 500 )
		};

		// Assuming you have a DSP preset created for Reverb
		// dspVolume.Dsp = ...
	}
}
```

### Setting up a Reverb Zone
1. Attach a `DspVolume` component to an empty GameObject in your scene.
2. In the Inspector, adjust the **Scene Volume** settings to match the physical space of a large room or cave.
3. Assign a DSP preset (like a Reverb effect) to the **Dsp** property.
4. Ensure the **Target Mixer** is set to "Game" or whichever mixer handles your game's sound effects.
5. When the player enters the volume, the game's audio will echo.

### Overlapping Volumes
If you have multiple overlapping `DspVolume` components (e.g., a large subtle reverb volume for an entire cave system, and a smaller intense reverb volume for a specific cavern), you can use the `Priority` property. The volume with the highest priority will be applied.

## Configuration

| Property | Description |
|:---|:---|
| `Dsp` | The `DspPresetHandle` to apply when the listener is inside the volume. |
| `TargetMixer` | The mixer channel to apply the DSP to (default is "Game"). |
| `Priority` | If multiple volumes overlap, the one with the highest priority wins. |
| `SceneVolume` | Inherited from `VolumeComponent`. Defines the shape (Box, Sphere, Capsule, Infinite) and size of the volume. |

## Troubleshooting

:::danger "The effect isn't turning on!"
1. Verify that your `Dsp` property actually has a valid preset assigned.
2. Ensure the `TargetMixer` matches the name of a real mixer in your audio setup (e.g., "Game").
3. Make sure the listener (usually your main Camera) is actually entering the bounds defined by `SceneVolume`.
:::

:::warning "Multiple effects are clashing"
Only the `DspVolume` with the highest `Priority` will be applied when overlapping. The engine will smoothly fade the mix between them, but you must set the priority correctly to determine which one takes precedence.
:::

## Related Pages
- [Audio Components](sound-components.md)
- [Playing Sounds (C#)](../../how-to/play-sounds.md)
