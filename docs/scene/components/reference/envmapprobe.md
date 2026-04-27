---
title: "Envmap Probe"
icon: "⭕"
sources:
  - engine/Sandbox.Engine/Scene/Components/Light/EnvmapProbe.cs
  - engine/Sandbox.Engine/Scene/Components/Light/EnvmapProbe.Baking.cs
updated: 2026-04-25
created: 2026-04-27
---

# Envmap Probe

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `EnvmapProbe` component.

An `EnvmapProbe` captures the environment around it as a cubemap to provide reflections and ambient lighting for objects in your scene.

## Quick Working Example

```csharp
public class DynamicReflection : Component
{
    protected override void OnStart()
    {
        var probe = Components.GetOrCreate<EnvmapProbe>();
        
        // Set it to update dynamically at runtime
        probe.Mode = EnvmapProbe.EnvmapProbeMode.Realtime;
        probe.UpdateStrategy = EnvmapProbe.CubemapDynamicUpdate.TimeInterval;
        probe.DelayBetweenUpdates = 0.1f;
        
        // Set a bounds for the probe's influence
        probe.Bounds = BBox.FromPositionAndSize( 0, 1024 );
    }
}
```

## Usage Modes

You can configure how the probe gathers its image using the `Mode` property:

| Mode | Description |
|---|---|
| `Baked` | The cubemap is generated in the editor and saved as a static texture. It's very cheap to render, making it ideal for static rooms. |
| `Realtime` | The cubemap renders dynamically while the game is running. This is expensive but necessary if your lighting changes dynamically or for moving vehicles. |
| `CustomTexture` | Uses an explicitly assigned texture asset instead of capturing the scene. |

## Realtime Update Strategies

If you use `EnvmapProbeMode.Realtime`, you must choose when it updates via `UpdateStrategy`:

- `OnEnabled`: Captures once when the probe is turned on.
- `EveryFrame`: Updates constantly. **Very slow, avoid unless strictly necessary.**
- `FrameInterval`: Updates every X frames.
- `TimeInterval`: Updates every X seconds (configured via `DelayBetweenUpdates`).

## Configuration

| Property | Description |
|---|---|
| `Projection` | How the reflection is projected onto surrounding geometry. |
| `Bounds` | The physical volume this probe influences. |
| `Feathering` | Softens the edges of the probe's influence so it blends smoothly with neighboring probes. |
| `Priority` | If an object is inside multiple probe bounds, the one with the highest priority wins. |

## Troubleshooting

:::danger Realtime Performance Cost
Placing multiple `Realtime` probes that update `EveryFrame` will cripple your frame rate because each probe has to render the scene 6 extra times (once for each side of the cube). Always prefer `Baked` for static environments, or use `TimeInterval` for slow-updating dynamic probes.
:::
