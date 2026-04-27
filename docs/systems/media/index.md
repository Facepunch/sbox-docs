---
title: "Media"
icon: "💿"
created: 2026-04-22
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Systems/Render/Multimedia/
---

# Media

s&box supports video playback, audio streaming, music, voice chat, positional 3D audio, and video encoding.

See [Video](video.md) and [Audio](audio.md) for details on the streaming APIs. For in-game sound effects and Sound components, see [Audio (gameplay)](../audio/index.md).

## Codec & Format Support

### Video

| Codec | Formats | Encode | Notes |
|---|---|---|---|
| VP9 | .webm, .mp4 | Yes | |
| AV1 | .webm, .mp4 | Yes | Best compression and quality |
| WebP | .webp | Yes | Supports transparency |
| H.264 | .mp4 | No | Windows only via Media Foundation, not recommended |

### Audio

| Codec | Formats | Notes |
|---|---|---|
| Opus | .ogg, .opus, .webm | |
| Vorbis | .ogg, .webm | |
| FLAC | .flac | |
| MP3 | .mp3 | |
| WAV / PCM | .wav | |
| AAC | .mp4 | Windows only via Media Foundation, not recommended |

:::warning
H.264 and AAC are only available on Windows via system decoders. Use VP9 or AV1 for video, and Opus, MP3, FLAC, or WAV for audio.
:::
