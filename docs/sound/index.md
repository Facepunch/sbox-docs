---
title: "Sound"
icon: "🔊"
created: 2026-04-22
updated: 2026-04-22
---

# Sound

s&box has a full audio system including 2D/3D sound playback, voice chat, positional audio, an audio effects pipeline, and lip sync support.

For music and programmatic audio streaming, see [Audio](../media/audio.md).

## Components

Audio components can be added to any GameObject in the editor. They handle everything from positional sounds and soundscapes to voice chat and lip sync.

| Component | Description |
|---|---|
| `SoundPointComponent` | Plays a sound at a point in the world |
| `SoundBoxComponent` | Plays a sound within a box area |
| `SoundscapeTrigger` | Plays a soundscape when the listener enters the trigger area |
| `AudioListener` | Overrides where the client hears audio from, instead of the camera |
| `LipSyncComponent` | Drives morphs with lipsync from sounds |
| `VoiceComponent` | Records and transmits microphone input to other players |

## In This Section

- [Playing Sounds](/sound/playing-sounds.md) — `Sound.Play`, `SoundHandle`, `GameObject.PlaySound`, and audio components
