---
title: "Hitboxes From Model"
icon: "🧠"
sources:
  - engine/Sandbox.Engine/Scene/Components/Game/ModelHitboxes.cs
updated: 2026-04-25
created: 2026-04-27
---

# Hitboxes From Model

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `ModelHitboxes` component.

`ModelHitboxes` builds a `Hitbox` for every entry in a `Model`'s `HitboxSet`, attached to the corresponding bones of a `SkinnedModelRenderer`. This is the standard way to give animated characters per-bone hitboxes that follow their skeleton — head, limbs, torso, etc., each tagged separately so a damage system can apply multipliers per region.

## Quick Working Example

```csharp
var citizen = Scene.CreateObject();
var renderer = citizen.AddComponent<SkinnedModelRenderer>();
renderer.Model = Model.Load( "models/citizen/citizen.vmdl" );

var hitboxes = citizen.AddComponent<ModelHitboxes>();
hitboxes.Renderer = renderer;
```

A trace can now hit the citizen's bones individually:

```csharp
var tr = Scene.Trace.Ray( ray, 1024f )
	.HitTriggers()
	.Run();

if ( tr.Hit && tr.Hitbox?.Tags.Has( "head" ) == true )
	damage *= 2f; // headshot
```

## Properties

| Property | Description |
|:---|:---|
| `Renderer` | The `SkinnedModelRenderer` providing the model and skeleton. Must reference a model with a populated `HitboxSet` for hitboxes to be created. |
| `Target` | The `GameObject` reported in trace hits. Falls back to this component's `GameObject`. |

## Common Patterns

### Updating positions per tick

Hitbox bodies don't track bones automatically. Call `UpdatePositions()` each tick (or fixed update, depending on your trace cadence) so traces see up-to-date positions:

```csharp
protected override void OnFixedUpdate()
{
	hitboxes.UpdatePositions();
}
```

In s&box's standard player setups this is already wired up by other systems.

### Re-tagging at runtime

Setting `GameObject.Tags` triggers `OnTagsChanged`, which copies the new tags onto every underlying physics shape — useful for switching the whole hierarchy between teams or states.

### Manual rebuild

If you change the model's `HitboxSet` at runtime (rare), call `Rebuild()` to regenerate the hitboxes.

:::warning "No hitboxes if the model has none"
If the model has an empty `HitboxSet`, no hitboxes are created — the component does nothing. The model must have hitboxes baked in via the model editor.
:::

:::note "Hitbox entries with no bone are skipped"
Any `HitboxSet` entry whose `Bone` is null is silently skipped during build.
:::

## Related Pages

- [Manual Hitbox](manualhitbox.md)
- [Skinned Model Renderer](skinnedmodelrenderer.md)
- [Hitboxes](hitboxes.md)
