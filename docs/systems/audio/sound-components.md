---
title: "Audio Components"
icon: "🔊"
sources:
  - engine/Sandbox.Engine/Scene/Components/Audio/BaseSoundComponent.cs
  - engine/Sandbox.Engine/Scene/Components/Audio/SoundPointComponent.cs
  - engine/Sandbox.Engine/Scene/Components/Audio/SoundBoxComponent.cs
  - engine/Sandbox.Engine/Scene/Components/Audio/AudioListener.cs
  - engine/Sandbox.Engine/Scene/Components/Audio/SoundscapeTrigger.cs
  - engine/Sandbox.Engine/Scene/Components/Audio/VoiceComponent.cs
  - engine/Sandbox.Engine/Scene/Components/Audio/LipSyncComponent.cs
updated: 2026-04-25
created: 2026-04-27
---

# Audio Components

> **New to audio?** Read **[Audio](index.md)** first — that's the overview. This is the reference for the audio components; the audio landing covers the full picture.

Audio components allow you to attach sounds directly to GameObjects in your scene, making it easy to create ambient noise or sounds tied to moving objects.

## DSP Volume Component

The `DspVolume` component applies DSP effects to a target mixer when the listener is inside its bounds. It inherits from `VolumeComponent` to define its physical shape (Sphere, Box, Capsule, Infinite).

* **Use case:** Applying reverb when entering a large empty room, or a muffling effect when underwater.

## Sound Point Component

The `SoundPointComponent` plays a sound emanating from a single specific point in space (the position of the GameObject). 

* **Use case:** A radio playing music, a humming computer terminal, or a crackling campfire.
* As the player moves away from the GameObject, the sound gets quieter based on the attenuation settings of the `SoundEvent` (or you can override the distance in the component).

## Sound Box Component

The `SoundBoxComponent` plays a sound that seems to fill a 3D rectangular area. Rather than emanating from a single point, the sound's position dynamically shifts to be at the closest point on the box's surface to the listener.

* **Use case:** A long river flowing through a level, or a large server room filled with humming racks. 
* If the player walks alongside the river (the box), the sound follows them along the edge, making it feel like a massive source of noise rather than a single point.

## Soundscape Trigger Component

The `SoundscapeTrigger` plays a complex, multi-layered Soundscape when the listener enters the trigger area. It handles fading in and out looping sounds and periodically firing stings (one-off sounds) randomly.

* **Use case:** Transitioning from the quiet echo of a cave to the bustling noise of a busy street.
* You can configure it as a Point (heard anywhere), Sphere (heard within a radius), or Box (heard within bounds).

## Voice Transmitter Component

The `Voice` component records and transmits voice/microphone input from the local player to other players over the network. 

* **Use case:** In-game voice chat, walkie-talkies, or proximity chat.
* It supports multiple activation modes: Always On, Push To Talk, and Manual (via the `IsListening` property).

```csharp
public class VoiceChatManager : Component
{
	[Property] public Voice VoiceComponent { get; set; }

	protected override void OnUpdate()
	{
		// Manually control listening state
		if ( Input.Pressed( "voice" ) )
		{
			VoiceComponent.IsListening = true;
		}
		else if ( Input.Released( "voice" ) )
		{
			VoiceComponent.IsListening = false;
		}
	}
}
```

## Lip Syncing Component

The `LipSync` component automatically drives the facial morphs (visemes) of a 3D character based on the currently playing audio or voice data.

* **Use case:** Animating a character's mouth while they talk over voice chat or play a recorded dialogue line.
* It works by reading the viseme weights from the `Sound` component and applying them to the `SkinnedModelRenderer` component. Ensure your model has the correct `viseme_` morph targets set up.

## Audio Listener Component

The `AudioListener` component determines the "ears" of the player in the 3D world. By default, the active `CameraComponent` acts as the listener.

If you add an `AudioListener` to a different GameObject, the engine will use that object's position to calculate how loud and from what direction 3D sounds should be heard.

* **Use case:** A security camera system where the player views a monitor, but you want them to hear the audio from the camera's location, not their physical body's location.

### Configuration (AudioListener)

| Property | Description |
|:---|:---|
| `UseCameraDirection` | If true, the listener uses the position of the GameObject, but the *rotation* (which way the ears are facing) still comes from the active Camera. This is crucial for keeping stereo audio aligned with what the player is looking at. |

## Common Configuration (Sound Components)

Both `SoundPointComponent` and `SoundBoxComponent` share these common properties:

| Property | Description |
|:---|:---|
| `SoundEvent` | The audio asset to play. |
| `TargetMixer` | The mixer channel to route this sound to. |
| `PlayOnStart` | Should the sound automatically start when the component is enabled? |
| `StopOnNew` | If true, when commanded to start again, it will immediately cut off the previous instance. |
| `Repeat` | If true, the sound will continuously restart after it finishes, waiting a random time between `MinRepeatTime` and `MaxRepeatTime`. |

### Overrides
You can enable override toggles on the component to ignore the settings defined in the `SoundEvent` asset:

* **Sound Override:** Override Volume, Pitch, or force it to be a 2D sound.
* **Distance Attenuation Override:** Manually set the max distance and falloff curve.
* **Occlusion Override:** Change if the sound is muffled by walls, and how thick those walls need to be (`OcclusionRadius`).

## Troubleshooting

:::warning Sounds Cutting Off Abruptly
If you spawn a GameObject with a `SoundPointComponent` and a `TemporaryEffect` component (e.g., for an explosion), make sure the GameObject isn't destroyed immediately. `TemporaryEffect` will intelligently wait for the sound to finish before deleting the object, but if you manually call `GameObject.Destroy()`, the sound will immediately cut out.
:::

## Related Pages
- [Playing Sounds (C#)](../../how-to/play-sounds.md)
