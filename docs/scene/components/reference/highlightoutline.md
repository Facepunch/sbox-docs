---
title: "Highlight Outline"
icon: "💡"
sources:
  - engine/Sandbox.Engine/Scene/Components/PostProcessing/Effects/HighlightOutline.cs
updated: 2026-04-25
created: 2026-04-27
---

# Highlight Outline

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `HighlightOutline` component.

`HighlightOutline` draws a coloured outline around a `Renderer`. It is a marker component — by itself it does nothing visual. The actual rendering is performed by a `Highlight` post-process effect on the camera, which finds every enabled `HighlightOutline` and draws their renderers into the outline pass.

Use it for object pickup prompts, selection rings, "you can interact with this" cues, etc.

## Quick Working Example

```csharp
public class Pickup : Component, IInteractable
{
    HighlightOutline _outline;

    public void OnHover()
    {
        _outline ??= AddComponent<HighlightOutline>();
        _outline.Enabled = true;
        _outline.Color = Color.Yellow;
        _outline.ObscuredColor = Color.Yellow * 0.4f;
    }

    public void OnUnhover()
    {
        if ( _outline.IsValid() )
            _outline.Enabled = false;
    }
}
```

## Common Patterns

### Toggle on hover

Adding/removing the component every frame is fine — it's a marker. But for performance, add it once and toggle `Enabled`.

```csharp
outline.Enabled = isHovered;
```

### Outline only specific meshes

```csharp
var outline = AddComponent<HighlightOutline>();
outline.OverrideTargets = true;
outline.Targets = new List<Renderer> { GunBarrelRenderer };
```

### Show a "you can see them through walls" effect

Set `ObscuredColor` to a visible value (e.g., a desaturated red) and `Color` to a brighter one, so allies / enemies are dimly visible behind cover and highlighted clearly when in line of sight.

```csharp
outline.Color = Color.Red;
outline.ObscuredColor = Color.Red * 0.3f;
outline.InsideColor = Color.Transparent;
```

## Properties

| Property | Default | Description |
|---|---|---|
| `Material` | `null` | Optional custom outline material. If null, a generated material is used. |
| `Color` | `Color.White` | Outline color when the renderer is visible. |
| `ObscuredColor` | `Color.Black * 0.4` | Outline color where the renderer is occluded by other geometry. |
| `InsideColor` | `Color.Transparent` | Fill color inside the outline (visible region). |
| `InsideObscuredColor` | `Color.Transparent` | Fill color inside the outline (obscured region). |
| `Width` | `0.25` | Width of the outline line. |
| `OverrideTargets` | `false` | If true, only `Targets` are outlined; otherwise every enabled descendant `Renderer` is outlined. |
| `Targets` | `null` | Explicit list of `Renderer`s when `OverrideTargets` is true. |

## Methods

- `GetOutlineTargets()` — returns the renderers the highlight effect should draw. Honors `OverrideTargets`.

:::warning "Outline doesn't show"
A `HighlightOutline` component does nothing on its own. Add a `Highlight` component to your camera so the outline pass is actually drawn.
:::

:::tip "Hidden hierarchies"
With `OverrideTargets = false`, only renderers under `FindMode.EnabledInSelfAndDescendants` are picked up — disabled renderers are ignored. Toggle the renderer's `Enabled` to remove it from the outline set without destroying the component.
:::

## Related Pages

- [Camera Component](cameracomponent.md)
- [Model Renderer](modelrenderer.md)
