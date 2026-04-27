---
title: "Sandbox.Engine: Audio"
icon: "🔊"
sources:
  - engine/Sandbox.Engine/Systems/Audio/AudioEngine.cs
  - engine/Sandbox.Engine/Systems/Audio/MixingThread.cs
  - engine/Sandbox.Engine/Systems/Audio/Mixer.cs
  - engine/Sandbox.Engine/Systems/Audio/SoundHandle.cs
created: 2026-04-27
updated: 2026-04-27
---

# Audio

The `Audio` subsystem manages all low-level sound mixing, routing, occlusion, and 3D spatialization within `Sandbox.Engine`.

## Architecture

At the heart of the engine's audio implementation is a hierarchical mixing system running asynchronously.

*   **`AudioEngine`**: The global static entry point. It manages device selection, total voice limits, and initializes the root mixers.
*   **`Mixer`**: A node in the audio routing graph. Mixers hold a volume, effects, and output to a parent mixer. The root is the "Master" mixer. `MultiChannelBuffer`s represent the actual data flowing through these nodes.
*   **`SoundHandle`**: An actively playing sound. Handles pitch bending, distance attenuation, and lip-sync buffer processing.
*   **`MixingThread`**: A dedicated background thread. It runs a tight loop invoking `AudioEngine.Mix()` which collects samples from `SoundHandle`s, runs them through the `Mixer` graph, and hands the final buffers to the hardware.

## The Mixer Graph

The engine creates a default set of mixers at startup (e.g., "Master", "Music", "UI", "Game"). Sounds are assigned to a mixer by their data definition or code.

During `AudioEngine.Mix()` (invoked via `MixingThread.MixOneBuffer()`):
1. The background thread safely locks its process to avoid interference with the main thread.
2. `Mixer.Master.StartMixing` is called on the Master mixer, cascading down to clear output buffers for active `Listener`s.
3. **Sampling:** `MixingThread.SampleVoices` executes a `Parallel.ForEach` over all active `SoundHandle`s. This efficiently reads the raw sound data required for the tick using underlying `AudioSampler` instances without blocking the main render loops.
4. `Mixer.MixVoices` gathers all active sampled `SoundHandle`s targeting that mixer.
5. **Binaural processing** is applied (via `SteamAudioSource` interop) to convert mono/stereo sounds into spatialized 3D audio for each active listener.
6. The processed voices are blended into the mixer's `MultiChannelBuffer`.
7. Child mixer buffers are mixed into parent buffers, ultimately culminating at the Master mixer, which scales the `MultiChannelBuffer` by the `settings.MasterVolume` and sends it to output.
8. **Disposal & Cleanup:** `MixingThread.FinishVoices` gracefully stops sounds that have finished playing or faded out, and enqueues their `AudioSampler`s to a `ConcurrentQueue` for thread-safe destruction between frames.

## 3D Spatialization & Steam Audio

When a `SoundHandle` has a world position, the `Mixer` attempts to apply spatialization using Steam Audio:

```csharp
// Internals from Mixer.cs
void ConvertToBinaural( SteamAudioSource source, Listener listener, SoundHandle voice, MultiChannelBuffer inputoutput )
{
    // If it's a UI sound, minimal processing is used
    if ( voice.ListenLocal )
    {
        source.ApplyBinauralMix( pos, spacial, useNearestInterpolation: true, _input, inputoutput );
        return;
    }

    // Convert world position relative to the listener's ear transform
    var soundDirectionLocal = listener.MixTransform.PointToLocal( voice.Position );
    
    // Send to Steam Audio's HRTF processor
    source.ApplyBinauralMix( soundDirectionLocal, voice.SpacialBlend, useNearest, _input, inputoutput );
}
```

*   **Thread Safety:** The `AudioEngine` makes heavy use of `Interlocked` reads/writes for volume floats because the `MixingThread` runs concurrently with the game logic (Scene `Update()`).
*   **Memory Allocations:** `MultiChannelBuffer` and samples arrays are aggressively reused. `MixVoices` sorts voices by age and culls those furthest away if the engine exceeds its maximum voice count without allocating arrays.
