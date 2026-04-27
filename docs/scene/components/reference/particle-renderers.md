---
title: "Particle Renderers"
icon: "📂"
sources:
  - engine/Sandbox.Engine/Scene/Components/Particles/Renderers/ParticleModelRenderer.cs
  - engine/Sandbox.Engine/Scene/Components/Particles/Renderers/ParticleSpriteRenderer.cs
  - engine/Sandbox.Engine/Scene/Components/Particles/Renderers/ParticleTextRenderer.cs
  - engine/Sandbox.Engine/Scene/Components/Particles/Renderers/ParticleTrailRenderer.cs
updated: 2026-04-27
created: 2026-04-27
---

# Particle Renderers

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the particle renderer components.

A particle effect in s&box is built from a `ParticleEffect` (which creates and updates particles) plus one or more **renderer** components that decide what each particle actually looks like. The four built-in renderer components — `ParticleSpriteRenderer`, `ParticleModelRenderer`, `ParticleTextRenderer`, and `ParticleTrailRenderer` — are added as siblings or children of the particle source and listen to its particles via `OnParticleCreated`.

## ParticleSpriteRenderer

Renders each particle as a 2D billboarded sprite from a `Sprite` resource. Supports static or animated sprites and is the workhorse for smoke, sparks, magic effects, etc.

| Property | Description |
|:---|:---|
| `Sprite` | The `Sprite` resource. May contain multiple animations. |
| `StartingAnimationName` | Animation to use when the scene starts. |
| `PlaybackSpeed` | Animation playback speed multiplier. |
| `Scale` | Sprite scale multiplier (`0..2`). Default `1`. |
| `Additive` | Render with additive blending. |
| `Shadows` | Cast shadows. |
| `Lighting` | Light the sprite from scene lighting. |
| `Opaque` | Skip transparency sorting (faster). |
| `TextureFilter` | Filter mode (`Bilinear` or `Point` for pixel art). |
| `Alignment` | Billboard mode: `LookAtCamera`, `RotateToCamera`, `Particle`, `Object`. |
| `SortMode` | `Unsorted` or `ByDistance`. |
| `DepthFeather` | Soft-intersection feathering against world geometry (`0..50`). |
| `FogStrength` | How much scene fog affects this sprite (`0..1`). |
| `FaceVelocity` | Aligns sprite to its velocity direction. |
| `RotationOffset` | Rotation offset added when `FaceVelocity`. |
| `MotionBlur` / `LeadingTrail` / `BlurAmount` / `BlurSpacing` / `BlurOpacity` | Optional motion-blur shape along the velocity vector. |

## ParticleModelRenderer

Renders each particle as a 3D model — useful for debris, gibs, or any volumetric particle.

| Property | Description |
|:---|:---|
| `Choices` | List of `ModelEntry` items (model + material group + body groups). One is picked at random per particle. |
| `MaterialOverride` | Material applied on top of every spawned model. |
| `RotateWithGameObject` | If true, the particle's rotation is composed with the `GameObject`'s world rotation. |
| `Scale` | Animatable scale multiplier (`ParticleFloat`). |
| `CastShadows` | Whether the spawned models cast shadows. |
| `RenderOptions` | Advanced render-options bag (folded by default). |

The legacy `Models` list is `[Hide]`-marked and obsolete — use `Choices`. A JSON upgrader migrates old data automatically.

## ParticleTextRenderer

Renders each particle as a billboarded text label, using a `TextRendering.Scope` to describe the text. Useful for damage numbers, in-world UI, debug overlays.

The rendering / billboard / motion-blur / face-velocity properties match `ParticleSpriteRenderer`. Text-specific:

| Property | Description |
|:---|:---|
| `Text` | The `TextRendering.Scope` (text, font, color, size, etc.). |
| `Pivot` | Normalised pivot for placement (`0..1`, default `0.5`). |
| `Scale` | Sprite scale (`0..2`, default `1`). |
| `Alignment` | `BillboardAlignment` shared with `ParticleSpriteRenderer`. |

### Changing the text from code

`Text` is a **`TextRendering.Scope` struct**, not a string. The struct holds the text content (`Scope.Text`) plus font, color, size, outlines, shadows, etc. Two pitfalls:

- `renderer.Text = "Hello"` doesn't compile — `Text` isn't a string.
- `renderer.Text.Text = "Hello"` *does* compile, but silently does nothing. Reading `renderer.Text` returns a copy of the struct; you'd be mutating a temporary.

The correct idiom is **read, mutate, write back**:

```csharp
var renderer = damage.GetComponent<ParticleTextRenderer>();

var scope = renderer.Text;          // copy out
scope.Text = $"-{damageAmount}";    // mutate the copy
scope.TextColor = Color.Red;        // (any other Scope field can be changed the same way)
scope.FontSize = 32;
renderer.Text = scope;              // assign back to push the change
```

Same pattern works for any `Scope` field: `FontName`, `FontWeight`, `Outline`, `Shadow`, `LetterSpacing`, etc.

## ParticleTrailRenderer

Renders a continuous trail behind each particle's path. The trail keeps building points as the particle moves and is automatically "adopted" (kept alive briefly) when the particle dies, so the trail can fade out cleanly rather than vanishing.

| Property | Description |
|:---|:---|
| `MaxPoints` | Maximum points stored per trail. Default `64`. |
| `PointDistance` | Minimum distance between trail points. Default `8`. |
| `LifeTime` | How long each trail point lives. Default `2`. |
| `Texturing` | `TrailTextureConfig` (material, texture, filter, address mode). |
| `Color` | `Gradient` along the trail length. |
| `Width` | `Curve` controlling trail width along its length. |
| `TintFromParticle` | Use the particle's color as the trail tint. |
| `ScaleFromParticle` | Use the particle's size as the trail scale. |
| `Wireframe` | Render in wireframe (debug). |
| `Opaque` | Treat as opaque (no blend mode, faster). |
| `CastShadows` | Cast shadows (only when `Opaque`). |
| `BlendMode` | `BlendMode` when not opaque. |
| `RenderOptions` | Advanced render-options bag. |

## Common Patterns

### Multiple renderers on one effect

Add a `ParticleSpriteRenderer` plus a `ParticleTrailRenderer` to the same particle source for a sparkle-with-trail effect. Each renderer independently spawns its own scene objects per particle.

### Picking a random model

`ParticleModelRenderer.Choices` is sampled at particle birth via `Random.Shared.FromList` — populate it with multiple `ModelEntry` items to get visual variety without writing any code.

## Related Pages

- [Sprite Renderer](spriterenderer.md)
- [Trail Renderer](trailrenderer.md)
- [Text Renderer](textrenderer.md)
