---
title: "Voice Transmitter"
icon: "🎤"
sources:
  - engine/Sandbox.Engine/Scene/Components/Audio/VoiceComponent.cs
updated: 2026-04-25
created: 2026-04-27
---

# Voice Transmitter

> **New to audio?** This is the reference for the `Voice` component. The audio system as a whole is documented at **[Audio](../../../systems/audio/index.md)**.

`Voice` records the local microphone, transmits the compressed audio to other clients via an unreliable broadcast RPC, and plays it back as 3D or screen-space audio on receivers. It also drives lipsync visemes onto a `SkinnedModelRenderer`, so a talking character's mouth animates in real time without an extra component.

The class name in code is just `Voice` (the file is `VoiceComponent.cs`).

## Quick Working Example

```csharp
public class Player : Component
{
    [Property] public SkinnedModelRenderer Body { get; set; }

    protected override void OnStart()
    {
        var voice = AddComponent<Voice>();
        voice.Mode = Voice.ActivateMode.PushToTalk;
        voice.PushToTalkInput = "voice";

        // Animate the player's face from voice audio
        voice.LipSync = true;
        voice.Renderer = Body;
    }
}
```

## Common Patterns

### Filtering who hears whom

Override `ShouldHearVoice( Connection )` to mute specific players (team chat, blocklists). Override `ExcludeFilter()` to choose who the broadcast RPC targets — by default it goes to everyone except those returned by the filter.

```csharp
public class TeamVoice : Voice
{
    protected override IEnumerable<Connection> ExcludeFilter()
    {
        // Only same-team players hear me
        return Connection.All.Where( c => GetTeam( c ) != MyTeam );
    }
}
```

### Self-monitoring

Set `Loopback = true` to play the local mic back through the user's speakers — useful for "test your mic" UIs.

### Visualising speech

Read `Amplitude` (0–1 loudness) for VU meters or talking icons. `LaughterScore` returns a 0–1 estimate of how laugh-like the current frame is.

```csharp
var bar = voice.Amplitude;
icon.Visible = voice.IsRecording;
```

## Properties

| Property | Default | Description |
|---|---|---|
| `Volume` | `1.0` | Playback volume multiplier. |
| `Mode` | `AlwaysOn` | `AlwaysOn`, `PushToTalk`, or `Manual`. |
| `PushToTalkInput` | `"voice"` | Input action name used when `Mode == PushToTalk`. |
| `WorldspacePlayback` | `true` | If true, voice plays at the GameObject's `WorldPosition` with occlusion. If false, plays head-locked. |
| `Loopback` | `false` | Play the local user's own voice back through their speakers. |
| `LipSync` | `true` | Enable viseme processing on the playback sound handle. |
| `Renderer` | `null` | `SkinnedModelRenderer` driven by lipsync visemes. |
| `MorphScale` | `3.0` | Multiplier applied to viseme weights (0–5). |
| `MorphSmoothTime` | `0.1` | Smoothing time for morph weight transitions (0–1). |
| `VoiceMixer` | `default` | `MixerHandle` the voice plays through. |
| `Distance` | `15000` | Audible distance for spatial playback. |
| `Falloff` | (default curve) | Distance-falloff curve. |

## Read-only state

| Property | Description |
|---|---|
| `IsRecording` | True while the local mic is actively recording. |
| `IsListening` | True if the mic *should* be open right now (mode-dependent). Settable in `Manual` mode. |
| `Amplitude` | 0–1 loudness of the current playback frame. |
| `LaughterScore` | 0–1 laughter-likeness of the current frame. |
| `Visemes` | 15 viseme weights for the current frame (or empty if no audio). |
| `LastPlayed` | `RealTimeSince` voice data last arrived. |

:::warning "Headless servers don't play voice"
`Msg_Voice` returns early if `Application.IsHeadless` — the dedicated server forwards voice but doesn't play it. Lipsync only animates on clients.
:::

:::tip "One recorder at a time"
The component uses a static `singleRecorder` reference. Enabling a second `Voice` while another is recording can cause the first to stop transmitting — design your UI so only one local `Voice` is active at a time.
:::

## Related Pages

- [Lip Sync](lipsync.md)
- [Sound Point](soundpointcomponent.md)
- [Audio Listener](audiolistener.md)
