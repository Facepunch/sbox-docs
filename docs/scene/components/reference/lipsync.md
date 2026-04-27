---
title: "Lip Sync"
icon: "😊"
sources:
  - engine/Sandbox.Engine/Scene/Components/Audio/LipSyncComponent.cs
updated: 2026-04-25
created: 2026-04-27
---

# Lip Sync

> **New to audio?** This is the reference for the `LipSync` component. The audio system as a whole is documented at **[Audio](../../../systems/audio/index.md)**.

`LipSync` reads viseme (mouth shape) weights from a playing sound and drives the matching morph targets on a `SkinnedModelRenderer`. Use it to animate a character's mouth from voice acting clips, recorded dialogue, or any `BaseSoundComponent` that produces audio.

The class name in code is `LipSync` (the file is `LipSyncComponent.cs`).

## Quick Working Example

```csharp
public class TalkingNPC : Component
{
    [Property] public SoundPointComponent Sound { get; set; }
    [Property] public SkinnedModelRenderer Head { get; set; }

    protected override void OnStart()
    {
        var lip = AddComponent<LipSync>();
        lip.Sound = Sound;
        lip.Renderer = Head;
    }
}
```

## Common Patterns

### Hooking a one-shot dialog line

Spawn a `SoundPointComponent`, point a `LipSync` at it, and the mouth will animate for the duration of the line. When the sound finishes, morphs return to neutral.

### Softer / sharper lip motion

`MorphScale` is a multiplier on the blended viseme weight; `MorphSmoothTime` controls how quickly the morph chases its target via `MathX.ExponentialDecay`. Larger smoothing = slower, smoother mouth motion.

```csharp
lip.MorphScale = 2.0f;       // exaggerated shapes
lip.MorphSmoothTime = 0.05f; // snappier
```

### Lip sync from voice chat

For real-time voice from another player, prefer the `Voice` component — it has built-in lip sync wired to its own internal sound handle and adds laughter / amplitude readouts.

## Properties

| Property | Default | Description |
|---|---|---|
| `Sound` | `null` | The `BaseSoundComponent` whose audio drives the visemes. Must be playing for visemes to be produced. |
| `Renderer` | `null` | The `SkinnedModelRenderer` whose morph targets are written. The model must have viseme morphs set up. |
| `MorphScale` | `1.5` | Multiplier on the final viseme weight (range 0–5). |
| `MorphSmoothTime` | `0.1` | Smoothing time for morph weight transitions (range 0–1). Internally multiplied by `0.17`. |

:::warning "Mouth doesn't move"
The component silently does nothing if any of the following are missing:
- `Sound` is null or its underlying handle isn't valid (sound not playing).
- `Renderer` is null, or its `SceneModel` isn't valid.
- `model.MorphCount == 0` — the model has no morph targets baked in.
- The model lacks viseme morphs, in which case `GetVisemeMorph` returns 0 and morphs stay neutral.
:::

:::tip "Resetting morphs"
On disable, `LipSync` calls `Renderer.SceneModel.Morphs.ResetAll()` to wipe the mouth back to neutral pose — important if you swap dialog systems mid-scene.
:::

## Related Pages

- [Voice](voice.md)
- [Sound Point](soundpointcomponent.md)
- [Skinned Model Renderer](skinnedmodelrenderer.md)
