---
title: Deal and Take Damage
icon: "🖼️"
sources:
  - engine/Sandbox.Engine/Scene/Components/Markers/IDamageable.cs
  - engine/Sandbox.Engine/Scene/DamageInfo.cs
created: 2026-04-27
updated: 2026-04-27
---

# Deal and Take Damage

Learn how to use the built-in `IDamageable` interface and `DamageInfo` class to standardized how objects take damage.

## Quick Working Example

```csharp
using Sandbox;

// 1. Implementing IDamageable on a target
public sealed class TargetDummy : Component, Component.IDamageable
{
	[Property] public float Health { get; set; } = 100f;

	public void OnDamage( in DamageInfo damage )
	{
		Health -= damage.Damage;
		Log.Info( $"Ouch! Took {damage.Damage} damage from {damage.Attacker?.Name}. Health is now {Health}." );

		if ( Health <= 0 )
		{
			GameObject.Destroy();
		}
	}
}

// 2. Dealing damage from a weapon
public sealed class Weapon : Component
{
	protected override void OnUpdate()
	{
		if ( Input.Pressed( "Attack" ) )
		{
			// Fire a ray forward
			var tr = Scene.Trace.Ray( GameObject.WorldPosition, GameObject.WorldPosition + GameObject.WorldRotation.Forward * 1000f )
				.IgnoreGameObjectHierarchy( GameObject.Root )
				.Run();

			if ( tr.Hit && tr.GameObject != null )
			{
				// Construct a DamageInfo object
				var damageInfo = new DamageInfo( 25f, GameObject.Root, GameObject );
				
				// Send the damage event to any IDamageable components on the hit GameObject
				foreach ( var damageable in tr.GameObject.Components.GetAll<IDamageable>( FindMode.EnabledInSelfAndDescendants ) )
				{
					damageable.OnDamage( damageInfo );
				}
			}
		}
	}
}
```

### The DamageInfo Object

The `DamageInfo` class contains several properties that help the target figure out what happened. You can use its constructors for quick initialization, or set properties individually.

```csharp
var damage = new DamageInfo( 50f, myPlayerGo, myWeaponGo )
{
	Position = hitResult.HitPosition,
	Origin = myWeaponGo.WorldPosition,
	Shape = hitResult.Shape
};

// Add tags to categorize the damage
damage.Tags.Add("fire", "burn");
```

### Broadcasting Damage

Often, a `GameObject` might have multiple components that need to know when it takes damage (e.g., a `HealthComponent` to reduce HP, and a `BloodSplatterComponent` to spawn effects). Instead of calling `OnDamage` on a single component, you can broadcast it to all `IDamageable` components on the object and its children.

```csharp
var damage = new DamageInfo( 10f, attacker, weapon );

foreach(var damageable in targetGameObject.Components.GetAll<IDamageable>(FindMode.EnabledInSelfAndDescendants))
{
	damageable.OnDamage(damage);
}
```

Or, even easier, if you want to broadcast the damage to the *entire* scene (e.g. for a global effect), you can use the event system, though this is rare for direct damage. Usually, iterating the hit object's components as shown above is the preferred approach for targeted damage.

## Configuration

### DamageInfo Properties

| Property | Description |
|---|---|
| `Damage` | The raw amount of damage dealt (float). |
| `Attacker` | The `GameObject` that caused the damage (usually the player or enemy). |
| `Weapon` | The `GameObject` used to cause the damage (e.g., a sword or a trap). |
| `Position` | The `Vector3` point where the impact physically occurred. |
| `Origin` | The `Vector3` point where the attack started (e.g. the barrel of a gun). |
| `Tags` | A `TagSet` string collection categorizing the damage (e.g., `"fire"`, `"blunt"`, `"fall"`). |
| `Hitbox` | The specific `Hitbox` that was struck (useful for headshot multipliers). |
| `Shape` | The physics `PhysicsShape` that was struck. |

## Troubleshooting

:::warning My target isn't taking damage!
Make sure the component on the target actually implements `Component.IDamageable`. Also, check that your raycast or collision detection is returning the correct `GameObject`. If your trace hits a child object (like an arm or a shield), the `IDamageable` script might be on the parent. Using `FindMode.EnabledInSelfAndAncestors` might be necessary depending on your hierarchy.
:::

## Related Pages
- [Component Interfaces](../scene/components/component-interfaces/index.md)
- [Trace / Raycast](trace-raycast.md)
