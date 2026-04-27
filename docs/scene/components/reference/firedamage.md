---
title: "Fire Damage"
icon: "🔥"
sources:
  - engine/Sandbox.Engine/Scene/Components/Game/FireDamage.cs
updated: 2026-04-25
created: 2026-04-27
---

# Fire Damage

> **New to physics?** Physics overview is at **[Physics](../../../systems/physics/index.md)**. This page is the reference for the `FireDamage` component.

`FireDamage` applies periodic damage to any `IDamageable` on its `GameObject.Root`. Damage is tagged `"fire"` and `"burn"`, so receivers can filter for fire-specific behavior (resistance, immunity, ignition spreading, etc.).

## Quick Working Example

Igniting an object for five seconds:

```csharp
public class Ignite : Component
{
	public void Burn( GameObject target, float duration = 5f )
	{
		var fx = target.AddComponent<FireDamage>();
		fx.DamagePerSecond = 25f;

		// Schedule removal
		_ = RemoveAfter( fx, duration );
	}

	async Task RemoveAfter( FireDamage fx, float seconds )
	{
		await Task.DelaySeconds( seconds );
		fx?.Destroy();
	}
}
```

## Properties

| Property | Description |
|:---|:---|
| `DamagePerSecond` | The damage rate in points per second. Default `20`. The component ticks at a fixed `0.2`s interval, so each tick applies `DamagePerSecond * 0.2` damage. |

## Common Patterns

### Fire resistance

Receivers can check the `Tags` on the incoming `DamageInfo` and reduce or ignore damage:

```csharp
public void OnDamage( in DamageInfo info )
{
	if ( info.Tags.Has( "fire" ) && IsFireproof )
		return;

	Health -= info.Damage;
}
```

### Spawning visual effects

`FireDamage` only applies damage — it doesn't render flames. Pair it with a particle system (e.g. a `Decal` for scorch, a particle emitter for flames) on the same `GameObject` for a complete "on fire" effect.

:::note "Damage routes through GameObject.Root"
`FireDamage` runs the event on `GameObject.Root`, not on the immediate parent. Attaching the component to a deeply-nested child still applies damage to the entire root hierarchy.
:::

:::warning "Networked: only the owner ticks"
`IsProxy` instances skip the tick. This avoids double-counting damage but means the GameObject must have a network owner (or be unowned, in which case the host ticks).
:::

## Related Pages

- [Radius Damage](radiusdamage.md)
- [Trigger Hurt](triggerhurt.md)
