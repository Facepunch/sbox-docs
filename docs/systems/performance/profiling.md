---
title: "Profiling & Overlays"
icon: "⚡"
sources:
  - engine/Sandbox.Engine/Core/Diagnostics/Profiler/Profiler.cs
  - engine/Sandbox.Engine/Systems/Render/Debug/Profiler.cs
  - engine/Sandbox.Engine/Systems/Render/Debug/GpuProfiler.cs
created: 2026-04-27
updated: 2026-04-27
---

# Profiling & Overlays

Understanding where your game spends its processing time is critical for maintaining a smooth framerate. s&box provides built-in tools to measure both CPU and GPU performance.

## Quick Working Example

You can enable on-screen performance overlays directly via the in-game console using console variables.

Press `~` to open the console and type:
```bash
# Shows the CPU Main Thread Timings overlay
debug_overlay_profiler 1

# Shows the GPU Timings overlay
debug_overlay_gpu_profiler 1
```

### The Profiler Overlay (CPU)

When enabled, the CPU overlay lists various engine subsystems (and your own C# code) sorted by average execution time.

| Column | Description |
|---|---|
| **name** | The name of the subsystem or code block. |
| **avg** | The smoothed average time (in milliseconds) spent in this block per frame. |
| **max** | The peak time recorded recently. |
| **%** | The percentage of the total frame time this block consumes. |

If you see `Managed:` entries taking a significant percentage of time, it means your C# game logic is heavy. Look for expensive operations in `OnUpdate()` or `OnFixedUpdate()`.

### The GPU Profiler Overlay

The GPU overlay provides a breakdown of rendering passes.

Passes like `Depth`, `Lighting`, or `PostProcess` will appear here. If your GPU time is high:
- **High Lighting/Shadows:** You may have too many dynamic lights with shadows enabled.
- **High PostProcess:** You may be using expensive volumetric effects or complex screen-space reflections.

### Deep Profiling (Firefox Profiler)

For an incredibly detailed, microsecond-level breakdown of what your C# code is doing, you can record a profiling trace and view it in the browser.

1. Open the console and type `profiler` to start recording.
2. Play your game for a few seconds during the performance stutter.
3. Type `profiler` again to stop.
4. The engine will automatically upload the trace and open [profiler.firefox.com](https://profiler.firefox.com/) in your browser with your data.

*(Note: If you do not want your profile uploaded to the web, use the `profiler_no_upload` command instead.)*

## Troubleshooting

:::danger "My game lags but the CPU/GPU profilers show low milliseconds!"
If your total frame time is low (e.g., 5ms) but the game feels jittery, you may be experiencing garbage collection (GC) spikes. This happens when your C# code creates too many temporary objects (allocations) every frame, forcing the engine to pause and clean them up. Use deep profiling (`profiler` command) to find allocation hotspots.
:::

:::warning "debug_overlay_profiler isn't showing up!"
Ensure you are running the game in a mode where debug overlays are allowed. If you are connected to a dedicated server as a normal client, you may not have permission to view certain debug information.
:::

## Related Pages
- [Performance Optimization](index.md)
