---
title: "Soundscape Trigger"
icon: "🎧"
sources:
  - engine/Sandbox.Engine/Scene/Components/Audio/SoundscapeTrigger.cs
updated: 2026-04-25
created: 2026-04-27
---

# Soundscape Trigger

> **New to audio?** This is the reference for the `SoundscapeTrigger` component. The audio system as a whole is documented at **[Audio](../../../systems/audio/index.md)**.

The `SoundscapeTrigger` component plays a `Soundscape` resource when the audio listener is inside a defined area. Use it to drive ambient audio per-room or per-region — e.g. forest birds in one zone, dripping water in another, with the engine cross-fading between them as the player moves.

## Quick Working Example

```csharp
public class AreaSetup : Component
{
	[Property] public Soundscape ForestAmbience { get; set; }

	protected override void OnStart()
	{
		var trigger = GameObject.AddComponent<SoundscapeTrigger>();
		trigger.Type = SoundscapeTrigger.TriggerType.Sphere;
		trigger.Radius = 1024f;
		trigger.Soundscape = ForestAmbience;
		trigger.Volume = 0.6f;
	}
}
```

## Properties

| Property | Description |
|:---|:---|
| `Type` | `Point`, `Sphere`, or `Box`. Determines how listener inclusion is tested. |
| `Soundscape` | The `Soundscape` resource describing the looped and sting sounds to play. |
| `TargetMixer` | The `MixerHandle` to play the soundscape on. Existing entries get re-routed when this changes. |
| `StayActiveOnExit` | If true, the soundscape keeps playing after the listener leaves until another soundscape takes over. Default `true`. |
| `Volume` | Master volume for this trigger (0..1). Live entries are updated when this changes. |
| `Radius` | Sphere radius when `Type == Sphere`. Default `500`. |
| `BoxSize` | Local half-extents when `Type == Box`. Default `(50, 50, 50)`. |

## Common Patterns

### Layering soundscapes

Place broad `Point` triggers as defaults, then layer smaller `Sphere`/`Box` triggers in specific rooms or zones. The soundscape system picks the most relevant one for the listener; with `StayActiveOnExit` the audio fades smoothly between zones rather than cutting out.

### Switching mixers

Reading or writing `TargetMixer` at runtime (e.g. ducking ambience when dialogue plays) re-routes any currently-playing entries automatically — no need to restart the soundscape.

:::note "Listener position, not camera position"
Triggers test the engine's audio listener, not the camera. If your game has a separate `AudioListener` component (e.g. attached to the player), make sure it's positioned where you expect.
:::

## Related Pages

- [Sound Point Component](soundpointcomponent.md)
- [Audio Listener](audiolistener.md)
