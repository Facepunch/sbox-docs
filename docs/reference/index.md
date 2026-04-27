---
title: "API Reference"
icon: "📖"
updated: 2026-04-25
created: 2026-04-27
---

# API Reference

> **Looking for the signatures of `Sandbox.*` types, methods, and properties?**
> The canonical, auto-generated C# API reference is **[sbox.game/api](https://sbox.game/api)**. It's built from the XML doc comments in the engine source, so it's always current with what the engine actually exposes. Bookmark it.

This wiki doesn't try to duplicate that. What it *does* cover, that the auto-generated reference can't:

## What's here

### [Component Reference](../scene/components/reference/index.md)
Per-component pages with **curated guidance** — properties grouped by concern, gotchas, common patterns, "use this over that" decisions, and cross-references to related components. The auto-generated reference will tell you that `Decal.LifeTime` is a `ParticleFloat`; our [Decal page](../scene/components/reference/decal.md) tells you that `LifeTime = 0` means "lives forever" and that you need a `TemporaryEffect` sibling to actually clean up the GameObject.

### [Shader Reference](../systems/shaders/reference/index.md) and [Shader Classes](../systems/shaders/classes/index.md)
HLSL globals, vertex/pixel input structures, and the `#include`-able helper classes (`Fog`, `Light`, `Bindless`, `Depth`, `Normals`, etc.). sbox.game/api covers C#; the shader language layer is documented here because the auto-generator can't reach it.

### [How-To Recipes](../how-to/index.md), [Tutorials](../tutorials/index.md), [Engine Concepts](../engine-concepts/index.md)
Goal-oriented and learning-oriented content — what the API reference deliberately doesn't do.

## When to use which

| You want to... | Go to |
|---|---|
| Look up the signature of `GameObject.Clone` | [sbox.game/api](https://sbox.game/api) |
| See every property on `ModelRenderer` | [sbox.game/api](https://sbox.game/api) |
| Understand *when* to set `Decal.Transient` vs `LifeTime` | [Decal Component](../scene/components/reference/decal.md) |
| Know what `g_flTime` and friends are in HLSL | [Shader Reference](../systems/shaders/reference/global-variables.md) |
| Sample roughness from the G-Buffer in a custom shader | [Shaders → Classes → G-Buffer](../systems/shaders/classes/g-buffer.md) |
| Learn how to deal damage between components | [How-To → Deal Damage](../how-to/deal-damage.md) |
| Build a third-person controller from scratch | [Tutorial](../tutorials/build-a-third-person-controller.md) |

## Tip: editor autocomplete is faster than any web reference

For day-to-day lookups, typing `Sandbox.` or `GameObject.` in your IDE and reading the autocomplete + XML doc tooltips is faster than tabbing to a website. Both Visual Studio and Rider show the same `/// <summary>` text that powers sbox.game/api.
