---
title: "Manual Hitbox"
icon: "🧠"
sources:
  - engine/Sandbox.Engine/Scene/Components/Game/ManualHitbox.cs
updated: 2026-04-25
created: 2026-04-27
---

# Manual Hitbox

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `ManualHitbox` component.

`ManualHitbox` adds a single hitbox shape to a `GameObject` without going through a model's hitbox set. Use it when you need precise hitboxes on something that isn't (or isn't only) defined by a `SkinnedModelRenderer` — destructible props, simple AI volumes, vehicle damage zones, or anything that a model's built-in hitboxes don't cover.

## Quick Working Example

```csharp
var go = Scene.CreateObject();
go.WorldPosition = WorldPosition;

var hb = go.AddComponent<ManualHitbox>();
hb.Shape = ManualHitbox.HitboxShape.Sphere;
hb.Radius = 24f;
hb.HitboxTags = new TagSet { "head" };
```

## Properties

| Property | Description |
|:---|:---|
| `Target` | The `GameObject` reported as the hit object. Falls back to this component's `GameObject` if unset. |
| `Shape` | `Sphere`, `Capsule`, `Box`, or `Cylinder`. |
| `Radius` | Radius in world units. Hidden in the inspector when `Shape == Box`. Default `10`. |
| `CenterA` | Local-space position. For boxes this is the box center. For capsules/cylinders this is one endpoint. |
| `CenterB` | Local-space position. For boxes this is the box's full size. For capsules/cylinders this is the second endpoint. Hidden when `Shape == Sphere`. |
| `HitboxTags` | Tags applied to the resulting `Hitbox` (e.g. `"head"`, `"limb"`). Used to drive damage multipliers. |

## Common Patterns

### Following a moving object

`ManualHitbox` doesn't auto-track its `WorldTransform` per-frame — it sets the body transform once at `Rebuild` time. If the `GameObject` moves, call `UpdatePositions()` from a tick to push the new transform into the underlying body:

```csharp
protected override void OnFixedUpdate()
{
	hitbox.UpdatePositions();
}
```

### Re-tagging at runtime

Setting `GameObject.Tags` triggers `OnTagsChanged`, which rebuilds the hitbox and copies the new tags onto the underlying physics shape automatically.

:::warning "Box uses size, not extents"
For `Box` shape, `CenterB` is the **full size** (width / depth / height), not half-extents. The component internally divides by 2 when constructing the physics shape.
:::

:::note "Use ModelHitboxes for skeletal hitboxes"
If you want hitboxes that follow animated bones, use [Hitboxes From Model](modelhitboxes.md) instead — it sources its hitboxes from a model's `HitboxSet`.
:::

## Related Pages

- [Hitboxes From Model](modelhitboxes.md)
- [Hitboxes](hitboxes.md)
