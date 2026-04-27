---
title: Audio, Effects & Rendering
icon: "📊"
created: 2026-04-25
updated: 2026-04-25
---

# Audio, Effects & Rendering

Sounds not playing, particles invisible, post-processing not applying, materials looking wrong.

## Audio

### "My sound isn't playing"

Most often: no `SoundEvent` assigned, or the sound is 3D and out of the listener's earshot.

- **Check:** `SoundEvent` is assigned in the Inspector.
- **Check:** for 3D sounds, the listener (camera) is within range.
- **Check:** `Sound.Play()` returns null on headless servers — wrap in a null-conditional.

**Deep dive:** [Audio Index](../systems/audio/index.md), [Sound Components](../systems/audio/sound-components.md), [Play Sounds](../how-to/play-sounds.md).

### "Sounds Cutting Off Abruptly"

A `SoundPointComponent` GameObject got destroyed before the sound finished. `TemporaryEffect` knows to wait for sounds; manual destroys don't.

- **Fix:** add a `TemporaryEffect` component so destruction is delayed until audio completes.

### "Mixer Handle is null"

`Mixer.FindMixerByName` / `MixerHandle.Get` returned null — the mixer doesn't exist in your project settings.

- **Fix:** add the mixer in Project Settings → Audio → Mixers, then re-fetch.

**Deep dive:** [Mixer](../systems/audio/mixer.md).

### "3D sounds aren't spatializing"

The mixer they route through has `Spacializing = 0`. UI and Music mixers default to non-spatial.

- **Fix:** either route 3D sounds through a different mixer, or raise `Spacializing` on the mixer.

### "DSP Volume effect isn't turning on"

Either `Dsp` preset is unassigned, or `TargetMixer` doesn't match a real mixer name.

- **Check:** `Dsp` has a preset assigned.
- **Check:** `TargetMixer` matches an existing mixer (case-insensitive).

**Deep dive:** [DSP Volume](../systems/audio/dsp-volume.md).

### "Multiple DSP effects clashing"

Only the highest-`Priority` overlapping `DspVolume` applies.

- **Fix:** set explicit `Priority` to control which wins.

## Particles and effects

### "My particles aren't showing up"

A particle effect needs **three** components on one GameObject: a `ParticleEffect`, an Emitter (e.g., `ParticleBoxEmitter`), and a Renderer (e.g., `ParticleSpriteRenderer`).

- **Fix:** add all three. Without an emitter, no particles spawn. Without a renderer, they're invisible.

**Deep dive:** [Particle Effect](../systems/effects/particle-effect/index.md), [Spawn Particles](../how-to/spawn-particles.md).

### "Emitter not spawning anything"

`ParticleEffect` parent is missing, or `Rate`/`Burst` is 0.

- **Check:** GameObject also has a `ParticleEffect` component.
- **Check:** `Rate > 0` or a `Burst` is configured.

**Deep dive:** [Particle Emitters](../systems/effects/particle-effect/particleemitters.md).

### "My particles won't go away"

`OnStop` behavior on `ParticleEffect` isn't set to destroy the GameObject.

- **Fix:** set `OnStop` to **Destroy GameObject** (or attach a `TemporaryEffect`).

### "Particles don't follow the moving object"

You spawned the particle at a world position once; it's not parented.

- **Fix:** parent the cloned particle GameObject to the moving GameObject so it inherits transform.

### "Game crashes when spawning objects from a particle"

Calling `GameObject.Clone` from inside `OnParticleStep` crashes — the step runs on a worker thread.

- **Fix:** queue the spawn for the main thread, or trigger spawns from `OnUpdate` instead.

**Deep dive:** [Custom Particle Controller](../systems/effects/particle-effect/custom-particle-controller.md).

### "Particles are lagging the game"

High `MaxParticles` plus collision is expensive on CPU.

- **Fix:** lower `MaxParticles`, simplify or disable collision, or move to a GPU-friendly emitter setup.

## Beams, tracers, decals

### "My beam is invisible"

`Alpha`/`Brightness` is 0, or the assigned material's shader isn't compatible.

- **Check:** `Alpha > 0`, `Brightness > 0`.
- **Check:** material uses a line-renderer-compatible shader (e.g., `materials/default/default_line.shader`).

**Deep dive:** [Beam Effect](../systems/effects/beams/beameffect.md).

### "My tracer stays in the air forever"

`BeamEffect` doesn't destroy its GameObject when its lifetime ends.

- **Fix:** add `TemporaryEffect` to the tracer prefab.

### "My tracer instantly reaches its target instead of traveling"

`TravelBetweenPoints` isn't enabled.

- **Fix:** tick `TravelBetweenPoints` on the `BeamEffect`.

**Deep dive:** [Tracers](../systems/effects/beams/tracers.md).

### "Decal isn't appearing"

Projects along its local **+X (forward)** axis. If you placed it flat against a wall, it may be projecting into the air.

- **Fix:** rotate the Decal so its forward arrow points into the surface.
- **Check:** `Depth > 0` and `Decals` (the list) is non-empty.

**Deep dive:** [Decal Component](../systems/effects/decals/decal-component.md).

### "Spawned decals leak memory"

`Transient` only removes excess decals beyond a global limit; it doesn't delete the GameObject.

- **Fix:** use a `TemporaryEffect` on the spawned GameObject so the entire object is destroyed after its lifetime.

## Lighting and probes

### "Probes inside geometry cause black splats"

Indirect light probes ended up embedded in walls.

- **Fix:** change `InsideGeometryBehavior` to `Deactivate` (or `Snap` if you want them to escape).

**Deep dive:** [Indirect Light Volumes](../systems/effects/indirect-light-volumes.md).

### "Volumetric fog isn't lighting up"

The lights crossing the volume need to opt into volumetric interaction.

- **Fix:** set `FogMode` on relevant lights to interact with volumetrics.
- **Check:** `Strength > 0` on the volume.

**Deep dive:** [Volumetric Fog Volume](../systems/effects/volumetric-fog-volume.md).

### "Performance: huge volumetric fog area"

Volumetric fog cost scales with bounds × density.

- **Fix:** tighten `Bounds` to where it's actually needed.

### "Realtime EnvmapProbe tanking framerate"

A `Realtime` probe rendering `EveryFrame` re-renders the scene 6× per probe per frame.

- **Fix:** prefer `Baked` probes for static environments. Use realtime sparingly and at lower update intervals.

**Deep dive:** [Envmap Probe](../scene/components/reference/envmapprobe.md).

### "DirectionalLight position doesn't seem to matter"

Correct — only its **rotation** affects lighting. Position is irrelevant.

**Deep dive:** [Lights](../scene/components/reference/lights.md).

### "Performance: too many shadow-casting lights"

Each shadow-casting light adds a render cost.

- **Fix:** limit dynamic shadow-casters per area. Use baked lighting or non-shadowed fill lights.

## Materials and shaders

### "My material is pink/black checkered"

Shader compilation error.

- **Fix:** open the material/shader. Check the engine console for HLSL errors.
- **Common cause:** plugging a Vector4 output into a float input in Shader Graph.

**Deep dive:** [Editor Shader Graph](../editor/shader-graph.md), [Shader Graph](../systems/shader-graph/index.md).

### "Exposed shader graph properties don't show up in Material Editor"

The variable's **Expose** checkbox isn't ticked, or the constant has no name.

- **Fix:** tick **Expose** on the variable. Name the constant.

### "My shader graph variables aren't showing in Material Editor"

Same fix — give the Constant node a name. Unnamed constants are baked into the shader.

### "Sampler State macros (CreateTexture2D, Tex2D) — should I use them?"

Older DX9-style API, limited.

- **Fix:** use modern decoupled samplers. The older macros are still supported but not recommended.

**Deep dive:** [Sampler States](../systems/shaders/sampler-states.md).

### "HLSL Syntax Errors in C# Shader Node"

`GenerateCode()` returns raw HLSL. A missing semicolon kills the whole graph.

- **Fix:** check the Output panel at the bottom of the Shader Graph editor for the precise error.

**Deep dive:** [Custom Shader Nodes](../systems/shader-graph/custom-nodes/index.md).

### "Missing Include Files (g_flTime, PixelInput undefined)"

Globals and structs live in headers you must `#include`.

- **Fix:** include the right base headers at the top of your shader. See the reference.

**Deep dive:** [Shader Reference](../systems/shaders/reference/index.md).

## Post-processing

### "Post-processing effects aren't showing"

`CameraComponent.EnablePostProcessing` is off.

- **Fix:** tick **Enable Post Processing** on the camera.
- **If using a `PostProcessVolume`:** the camera (or `PostProcessAnchor`) must be inside the volume bounds.

**Deep dive:** [Post Processing](../systems/post-processing/index.md), [Effects](../systems/post-processing/effects/index.md).

### "Effect snaps on/off entering a Post Process Volume"

You're reading the property directly off the component. Use the weighted accessor so blending works.

- **Fix:** in your `Render()`, use `GetWeighted( x => x.MyProperty )` instead of direct access.

**Deep dive:** [Creating Post Processes](../systems/post-processing/creating-postprocesses.md).

### "Post Process Volume doesn't apply effects"

`SceneVolume` isn't configured (Type, Bounds), or no effects are attached as children.

- **Fix:** set `SceneVolume.Type` (Box, Sphere, Capsule) and proper bounds. Attach effect components.

**Deep dive:** [Post Process Volume](../systems/post-processing/postprocessvolume.md).

### "Bloom isn't blooming"

`Threshold` too high, or your scene has no bright pixels above the threshold.

- **Fix:** lower `Threshold`. Or add an emissive material with `Emission > 1`.

**Deep dive:** [Bloom](../systems/post-processing/effects/bloom.md).

### "Whole screen is glowing from bloom"

`Threshold` too low — normal lit surfaces are above it.

- **Fix:** raise `Threshold` to ~1.0–1.5 so only HDR-bright sources bloom.

### "Depth of Field tanks framerate"

DoF requires multiple blurry sample passes.

- **Fix:** lower `BlurSize`, or set `r_dof_quality` lower in console.

**Deep dive:** [Depth of Field](../systems/post-processing/effects/depthoffield.md).

### "Everything is blurry from DoF"

`FocalDistance` is set where there's nothing in focus.

- **Fix:** raycast from the camera to find scene depth, set `FocalDistance` to the trace distance.

## Cameras

### "My screen is totally black"

No `CameraComponent` in the scene, or the camera GameObject is disabled.

- **Fix:** add a `CameraComponent` somewhere. Make sure it's enabled.

**Deep dive:** [Camera Component](../scene/components/reference/cameracomponent.md), [Camera Follow](../how-to/camera-follow.md).

### "Z-fighting / flickering textures"

`ZNear` is too close to 0 — depth precision collapses.

- **Fix:** raise `ZNear` to something like `10` (or whatever your scene scale needs).

### "Camera stutters when following a Rigidbody"

Rigidbody updates in `OnFixedUpdate` (physics tick), camera in `OnUpdate` (render tick) — they don't align.

- **Fix:** move the camera in `OnLateUpdate` and read interpolated transform, or follow with smoothing.

## Cubemap / sky

### "CubemapFog: I see no fog"

`CubemapFog` requires a `CameraComponent` on the **same GameObject**.

- **Fix:** add a CameraComponent to the same GameObject.

**Deep dive:** [Cubemap Fog](../scene/components/reference/cubemapfog.md).

### "Cubemap fog looks pixelated / sharp"

Background image isn't blurred enough.

- **Fix:** raise `Blur`. Cubemap fog reads best when softly blurred.

### "Skybox material won't update from code"

The engine enforces that the assigned material's shader name contains `"sky"`.

- **Fix:** use a sky shader. Standard model shaders are silently rejected.

**Deep dive:** [Skybox 2D](../scene/components/reference/skybox2d.md).

### "Multiple GradientFog components — only one is taking effect"

There's one global gradient fog state per scene. Last to render wins.

- **Fix:** use only one active `GradientFog` at a time.

## Performance

### "Game stutters every few seconds"

Garbage Collection (GC) spike. Your code is allocating heavily on the hot path.

- **Fix:** use the profiler. Look for GC events in Firefox Profiler. Cache, reuse, and avoid LINQ/lambdas in `OnUpdate`.

**Deep dive:** [Profiling](../systems/performance/profiling.md), [Optimization Guide](../systems/performance/optimization-guide.md).

### "Frame rate drops looking at one specific object"

GPU-bound. Probably high poly count, expensive shader, or volumetric shadows on that object.

- **Fix:** profile GPU. Reduce poly count or shader complexity. Disable shadow casting for problematic objects.

### "debug_overlay_profiler isn't showing"

You're running in a mode that doesn't allow debug overlays (e.g., remote-connected to a non-development server).

- **Fix:** test in editor or local server, where the overlay is permitted.

## Related Pages

- [Scene & Components](./scene-and-components.md)
- [Editor](./editor.md)
- [Runtime & Build](./runtime-and-build.md)
