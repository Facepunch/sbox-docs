---
title: "Animated Effects"
icon: "🏃‍♂️"
created: 2025-06-26
updated: 2025-06-26
sources:
  - engine/Sandbox.Engine/Scene/Components/Render/Decal.cs
---

# Animated Effects

Several `Decal` properties aren't plain numbers — they're `ParticleFloat` or `ParticleGradient` curves evaluated over the decal's lifetime. This is the same animation system the [`ParticleEffect`](../particle-effect/index.md) component uses, so any pattern you've built there works here.

![Decal property animation](./images/decal-property-animation.mp4)

## Which properties animate

| Property | Type | Drives |
|---|---|---|
| `Scale` | `ParticleFloat` | Multiplier on `Size`. Default `1`. Animate it to grow or shrink the decal over time. |
| `Rotation` | `ParticleFloat` | Rotation in degrees. Default range `0..360` — every spawn rotates randomly so impacts don't tile. |
| `Parallax` | `ParticleFloat` | Parallax depth strength when a `HeightTexture` is set. Default `1`. |
| `ColorTint` | `ParticleGradient` | Albedo tint over lifetime — drop alpha at the end for a fade-out. Default `Color.White`. |
| `ColorMix` | `ParticleFloat` | `0..1` blend between surface colour and decal colour. Animate it to fade into a normal-only impression. |
| `Alpha` | `ParticleFloat` | Independent alpha multiplier. Cleaner than fading via `ColorTint` if you only want opacity to change. |

## Working with `ParticleFloat`

`ParticleFloat` lets each value be a constant, a random range, or a curve evaluated against the decal's lifetime fraction. In the inspector you'll see a dropdown to pick between the modes; in code:

```csharp
decal.Scale = new ParticleFloat( 0.8f, 1.2f );  // random per-spawn

decal.Rotation = new ParticleFloat
{
    Type = ParticleFloat.ValueType.Range,
    ConstantA = 0,
    ConstantB = 360
};
```

A common pattern: spawn a bullet hole that visually settles in over the first 100ms. Animate `Scale` from `1.5` down to `1.0` evaluated as a curve over lifetime.

## Working with `ParticleGradient`

`ColorTint` is a `ParticleGradient` — a gradient evaluated against lifetime fraction. The most useful pattern is a fade-out: keep the colour stops constant, drop the alpha at the end.

```csharp
// In code: usually you author the gradient in the inspector; this is what
// the data structure looks like if you do need to set it programmatically.
decal.ColorTint = Color.White.WithAlpha( 1.0f );  // simple non-animated tint
```

For animated fades, set up the gradient in the Inspector — the curve editor is faster than constructing one in code.

## Pairing with lifetime

Animation curves only have anything to evaluate against if the decal has a finite `LifeTime`. A static decal (`LifeTime = 0` or `Looped = true`) freezes at the curve's start position. See [Lifetime](lifetime.md) for the full lifecycle rules.
