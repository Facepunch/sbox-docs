---
title: "Video"
icon: "🎥"
created: 2026-04-22
updated: 2026-04-22
---

# Video

## VideoPlayer

Decodes a video file or URL and exposes the current frame as a `Texture`.

| Codec | Containers | Notes |
|---|---|---|
| VP9 | .webm, .mp4 | |
| AV1 | .webm, .mp4 | Best compression and quality |
| WebP | .webp | Supports transparency |
| H.264 | .mp4 | Windows only via Media Foundation, not recommended |

```csharp
var player = new VideoPlayer();
player.OnLoaded += () => Log.Info( $"Loaded {player.Width}x{player.Height}" );
player.Play( FileSystem.Mounted, "videos/intro.webm" );
```

Streaming over HTTP/HTTPS is also supported:

```csharp
player.Play( "https://example.com/stream.webm" );
```

`VideoPlayer.Texture` updates each frame and can be used anywhere a `Texture` is accepted.

```csharp
panel.Style.SetBackgroundImage( player.Texture );
material.Set( "Texture", player.Texture );
```

Audio settings live on the `Audio` accessor:

```csharp
player.Audio.Volume = 0.5f;
player.Audio.Position = WorldPosition;
player.Audio.TargetMixer = Mixer.FindMixerByName( "Music" );
```

## VideoWriter

Encodes a sequence of RGBA frames into a video file. Used internally by the screen recorder.

| Codec | Containers | Notes |
|---|---|---|
| VP9 | .webm, .mp4 | Best compatbility |
| AV1 | .webm, .mp4 | Best compression and quality |
| WebP | .webp | Animated WebP, supports transparency |

WebM supports transparency. The `EncodingPreset` on `VideoWriter.Config` controls speed vs quality: `Fast` for real-time recording, `Balanced` for general use, `Quality` for offline export.

```csharp
var writer = new VideoWriter( "recordings/output.mp4", new VideoWriter.Config
{
    Width = 1920,
    Height = 1080,
    FrameRate = 60,
    Bitrate = 14,
    Codec = VideoWriter.Codec.AV1,
    Container = VideoWriter.Container.MP4,
    Preset = VideoWriter.EncodingPreset.Fast,
    AudioCodec = VideoWriter.AudioCodec.Opus,
} );

writer.AddFrame( rgbaBytes );

await writer.FinishAsync();
```
