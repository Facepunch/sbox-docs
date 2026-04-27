---
title: "Hitboxes"
icon: "🧠"
sources:
  - engine/Sandbox.Engine/Scene/Components/Game/Hitbox.cs
  - engine/Sandbox.Engine/Scene/Components/Game/ManualHitbox.cs
  - engine/Sandbox.Engine/Scene/Components/Game/ModelHitboxes.cs
updated: 2026-04-25
created: 2026-04-27
---

# Hitboxes

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the hitbox components.

Define specific damage zones on a GameObject, either by creating manual shapes or by automatically importing them from a 3D model's skeleton.

## Quick Working Example

```csharp
using Sandbox;

public sealed class TargetDummy : Component, Component.IDamageable
{
	[RequireComponent] public ModelHitboxes Hitboxes { get; set; }

	[Property] public float Health { get; set; } = 100f;

	public void OnDamage( in DamageInfo damage )
	{
		float damageAmount = damage.Damage;

		// Check if the specific hitbox that was struck has the "head" tag
		if ( damage.Hitbox != null && damage.Hitbox.Tags.Has( "head" ) )
		{
			damageAmount *= 2f; // Headshot multiplier
			Log.Info( "Headshot!" );
		}

		Health -= damageAmount;
	}
}
```

## Components

s&box provides two ways to add hitboxes to your GameObjects:

1.  **Manual Hitbox:** Allows you to define a simple, specific shape (Sphere, Capsule, Box, or Cylinder) directly in the editor. Useful for simple props or creating custom invisible weak points.
2.  **Model Hitboxes:** Automatically generates a full set of hitboxes by reading the physics hull and tags embedded inside a 3D model (typically driven by a `SkinnedModelRenderer`). The hitboxes will automatically follow the model's skeletal animation.

### Detecting Specific Hit Zones

When dealing damage using a raycast, the resulting `DamageInfo` object will contain a reference to the `Hitbox` that was struck. You can use the `Tags` on that hitbox to determine if it was a headshot, an arm, or an armored section.

```csharp
// Dealing damage (usually in a weapon script)
var tr = Scene.Trace.Ray( startPos, endPos ).Run();

if ( tr.Hit && tr.GameObject != null )
{
	var damageInfo = new DamageInfo( 25f, GameObject.Root, GameObject )
	{
		Position = tr.HitPosition,
		Hitbox = tr.Hitbox // Pass the struck hitbox into the damage info
	};

	foreach ( var damageable in tr.GameObject.Components.GetAll<IDamageable>( FindMode.EnabledInSelfAndDescendants ) )
	{
		damageable.OnDamage( damageInfo );
	}
}
```

### Manual Hitbox Properties

| Property | Description |
|---|---|
| `Target` | The `GameObject` that will be reported as the owner when this hitbox is struck. If left empty, it defaults to the GameObject this component is attached to. |
| `Shape` | The physical shape of the hitbox (`Sphere`, `Capsule`, `Box`, or `Cylinder`). |
| `Radius` | The radius of the shape (applies to Sphere, Capsule, and Cylinder). |
| `CenterA` / `CenterB` | Defines the start and end points for the shape (or its bounds for a Box). |
| `HitboxTags` | A set of tags applied to this specific hitbox, useful for identifying weak points like `"head"` or `"glass"`. |

### Model Hitboxes Properties

| Property | Description |
|---|---|
| `Renderer` | The `SkinnedModelRenderer` component that holds the model and skeleton you want to take the hitboxes from. |
| `Target` | The `GameObject` that will be reported as the owner when a hitbox is struck. Defaults to this GameObject. |

## Troubleshooting

:::danger "My ModelHitboxes aren't showing up or following my animation!"
Ensure that the `Renderer` property on your `ModelHitboxes` component is assigned to a valid `SkinnedModelRenderer` on your GameObject. Also, verify that the 3D model asset itself actually has hitboxes defined in the Model Editor.
:::

:::warning "Trace hits aren't detecting my hitbox!"
Make sure your trace is configured to hit the correct physics tags. By default, `ManualHitbox` and `ModelHitboxes` inherit the `Tags` of their parent `GameObject`. If your GameObject is missing the necessary tags (like `"player"` or `"solid"`), the trace might ignore the hitbox entirely.
:::

## Related Pages
- [Deal and Take Damage](../../../how-to/deal-damage.md)
- [Trace / Raycast](../../../systems/physics/physics-events.md)
- [Colliders](../../../systems/physics/colliders.md)
