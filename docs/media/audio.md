---
title: "Audio"
icon: "🔊"
created: 2026-04-22
updated: 2026-04-22
---

# Audio

## MusicPlayer

`MusicPlayer` is for music tracks and streaming audio that need programmatic control over playback, positioning, or visualization. For in-game sound effects and Sound components, see [Sound](../../sound/index.md).

```csharp
var music = MusicPlayer.Play( FileSystem.Mounted, "music/theme.ogg" );
music.Volume = 0.8f;
music.Repeat = true;
```

Streaming from a URL:

```csharp
var music = MusicPlayer.PlayUrl( "https://example.com/stream.ogg" );
```

By default, `MusicPlayer` plays globally. To place it in world-space:

```csharp
music.ListenLocal = false;
music.Position = WorldPosition;
```

Audio visualization:

```csharp
ReadOnlySpan<float> spectrum = music.Spectrum;   // 512-element FFT magnitudes
float amplitude = music.Amplitude;               // approximate loudness
```

### Supported Formats

| Codec | Containers |
|---|---|
| Opus | .ogg, .opus, .webm |
| Vorbis | .ogg, .webm |
| FLAC | .flac |
| MP3 | .mp3 |
| WAV / PCM | .wav |
| AAC | MP4 (Windows only via Media Foundation, not recommended) |

:::warning
AAC is only available on Windows via system decoders and is not recommended. Use Opus instead.
:::
