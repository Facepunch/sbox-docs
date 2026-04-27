---
title: "Radius Damage"
icon: "✨"
sources:
  - engine/Sandbox.Engine/Scene/Components/Game/RadiusDamage.cs
updated: 2026-04-25
created: 2026-04-27
---

# Radius Damage

> **New to physics?** Physics overview is at **[Physics](../../../systems/physics/index.md)**. This page is the reference for the `RadiusDamage` component.

The `RadiusDamage` component applies area-of-effect damage and physical force to surrounding objects, optionally checking for line-of-sight (occlusion).

## Quick Working Example

```csharp
using Sandbox;

public sealed class ExplosiveBarrel : Component, Component.IDamageable
{
	[RequireComponent] public RadiusDamage ExplosionRadius { get; set; }
	[Property] public float Health { get; set; } = 50f;
	[Property] public GameObject ExplosionEffectPrefab { get; set; }

	public void OnDamage( in DamageInfo damage )
	{
		Health -= damage.Damage;

		if ( Health <= 0 )
		{
			// Spawn visual effect
			if ( ExplosionEffectPrefab.IsValid() )
			{
				ExplosionEffectPrefab.Clone( WorldPosition );
			}

			// Apply the radius damage and physical force
			ExplosionRadius.Apply();

			// Destroy the barrel
			GameObject.Destroy();
		}
	}
}
```

### Exploding on Awake

If you have a missile or grenade prefab that has already "exploded" and is just being spawned into the world to deal its damage, you can set `DamageOnEnabled` to `true`. As soon as the `RadiusDamage` component becomes enabled (usually upon spawning), it will immediately apply its damage once.

### Triggering Manually

For environmental hazards like explosive barrels or spells, you usually leave `DamageOnEnabled` as `false` and manually call `Apply()` when the event happens.

```csharp
// Somewhere in your code
myRadiusDamage.Apply();
```

## Configuration

| Property | Description |
|:---|:---|
| `Radius` | The radius of the damage area in units. Default: `512`. |
| `PhysicsForceScale` | How much physics force should be applied to push objects away. Default: `1`. |
| `DamageOnEnabled` | If enabled, applies damage immediately as soon as the component is enabled. Default: `true`. |
| `Occlusion` | Should the world shield victims from damage? If true, it performs raycasts to ensure there is a clear line of sight. Default: `true`. |
| `DamageAmount` | The maximum amount of damage inflicted at the very center of the blast. Damage scales down towards the edge. Default: `100`. |
| `DamageTags` | Tags to apply to the damage invoice (e.g., `"explosive"`, `"fire"`), allowing the victim to react differently. |
| `Attacker` | The `GameObject` to credit with this attack (useful for kill feeds). |

## Troubleshooting

:::danger "Objects behind walls are taking damage!"
Check if `Occlusion` is checked. If `Occlusion` is true and objects behind walls are still taking damage, ensure your wall geometry has proper physics collision and does not have tags like "trigger", "gib", or "debris" which the occlusion trace ignores.
:::

:::warning "No physics force is being applied!"
The `RadiusDamage` component applies force using `Rigidbody.ApplyForceAt()`. Ensure the target objects have a `Rigidbody` component and that their `MotionEnabled` property is true (they aren't static/kinematic).
:::

## Related Pages
- [Deal and Take Damage](../../../how-to/deal-damage.md)
- [Hitboxes](hitboxes.md)
