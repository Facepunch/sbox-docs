---
title: "IDamageable"
icon: "🤍"
created: 2026-04-25
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Scene/Components/Markers/IDamageable.cs
  - engine/Sandbox.Engine/Scene/DamageInfo.cs
---

# IDamageable

`Component.IDamageable` marks a component as something that can take damage. It's the standard contract every damage-dealing system in s&box looks for: hitscan weapons, explosions (`RadiusDamage`), trigger volumes (`TriggerHurt`, `FireDamage`), AI attacks. They don't care what your class is — they just walk `IDamageable`s in the affected area and call `OnDamage` on each one.

## Quick Working Example

```csharp
using Sandbox;

public sealed class PlayerHealth : Component, Component.IDamageable
{
    [Property, Sync] public float Health { get; set; } = 100f;
    [Property] public float MaxHealth { get; set; } = 100f;

    public void OnDamage( in DamageInfo damage )
    {
        if ( IsProxy ) return;        // only the owner mutates Health
        if ( Health <= 0 ) return;    // already dead

        Health = MathF.Max( 0, Health - damage.Damage );

        if ( damage.Tags.Has( "explosion" ) )
            Log.Info( $"Took {damage.Damage} explosion damage from {damage.Attacker?.Name}" );

        if ( Health <= 0 )
            OnDeath( damage );
    }

    void OnDeath( in DamageInfo damage )
    {
        // Award the kill, ragdoll, drop loot, respawn timer, etc.
    }
}
```

## Interface

```csharp
public interface IDamageable
{
    void OnDamage( in DamageInfo damage );
}
```

That's it. One method. Required. The `in` modifier is a perf hint (passes by readonly reference) — `DamageInfo` is a class, not a struct, so it wouldn't be copied either way, but the `in` makes the no-mutate intent explicit.

## The DamageInfo class

`DamageInfo` is **a class**, deliberately — games can derive their own subtypes (e.g. `FireDamageInfo : DamageInfo`) without the engine knowing about them.

| Field | Type | Purpose |
|---|---|---|
| `Attacker` | `GameObject` | Usually the player or NPC dealing damage. |
| `Weapon` | `GameObject` | The weapon, vehicle, or source object. |
| `Hitbox` | `Hitbox` | The specific hitbox that was hit, if any (head, leg, etc.). |
| `Damage` | `float` | Amount of damage. |
| `Origin` | `Vector3` | Where the damage came from — bullet shooter's eye, explosion centre. |
| `Position` | `Vector3` | Where on the hit object it landed. |
| `Shape` | `PhysicsShape` | The physics shape that took the hit. |
| `Tags` | `TagSet` | Damage type tags: `"explosion"`, `"bullet"`, `"fire"`, etc. Use these for armour systems and resistance logic. |

Constructors:

```csharp
new DamageInfo();
new DamageInfo( damage: 25f, attacker: shooter, weapon: gun );
new DamageInfo( damage: 25f, attacker: shooter, weapon: gun, hitbox: headHitbox );
```

Set anything else (Tags, Origin, Position, Shape) by property after construction.

> The legacy `DamageInfo.IsExplosion` boolean is `[Obsolete]` — use `damage.Tags.Has("explosion")` instead. The same Tags-based pattern handles every damage type.

## Common patterns

### Multiplayer authority

`OnDamage` runs on whichever machine sent the damage call. **Mutate networked state only on the owner** (or use `[Sync(SyncFlags.FromHost)]` for host-authoritative health):

```csharp
public void OnDamage( in DamageInfo damage )
{
    if ( IsProxy ) return;            // gate the mutation
    Health -= damage.Damage;
}
```

### Damage filtering by tag

```csharp
public void OnDamage( in DamageInfo damage )
{
    if ( damage.Tags.Has( "fire" ) && IsFireResistant ) return;
    if ( damage.Tags.Has( "explosion" ) ) Health -= damage.Damage * 1.5f; // weak to explosions
    else                                  Health -= damage.Damage;
}
```

### Hitbox multipliers

```csharp
public void OnDamage( in DamageInfo damage )
{
    var multiplier = damage.Hitbox?.Tags.Has( "head" ) == true ? 4f : 1f;
    Health -= damage.Damage * multiplier;
}
```

### Forwarding damage up the hierarchy

If your hitbox component is on a child but health lives on the root, forward the call:

```csharp
public void OnDamage( in DamageInfo damage )
{
    GameObject.Root.Components.Get<PlayerHealth>()?.OnDamage( damage );
}
```

## Discovering damageables

The standard way damage systems find targets:

```csharp
// Anywhere in the scene
foreach ( var d in Scene.GetAll<Component.IDamageable>() )
    d.OnDamage( damageInfo );

// Inside a sphere (RadiusDamage-style)
foreach ( var d in Scene.GetAllComponents<Component.IDamageable>() )
{
    if ( !(d is Component c) || c.GameObject is not { } go ) continue;
    if ( go.WorldPosition.Distance( origin ) > radius ) continue;
    d.OnDamage( damageInfo );
}

// Targeted to a specific GameObject (and its children)
gameObject.RunEvent<Component.IDamageable>( x => x.OnDamage( damageInfo ) );
```

The engine's built-in damage components (`RadiusDamage`, `TriggerHurt`, `FireDamage`) all use one of these patterns under the hood.

- **Damage can arrive on a destroyed component.** Always check `if ( !this.IsValid() ) return;` at the top of `OnDamage` if you queue it asynchronously, or guard the consumer.
- **Multiple `IDamageable`s on one GameObject.** Both are valid and both will receive the call when a sphere/trigger query hits the GameObject. Decide explicitly which one owns "health" — usually the root one.
- **Negative damage.** The interface doesn't validate `Damage`. Some games use negative damage for healing. If you don't want to support that, clamp at 0 in your handler.
- **Don't subscribe to `IDamageable` at the system level for healing or buffs.** It's specifically for damage. Make a parallel `IHealable` if you need that pattern.

## See also

- [Component Interfaces overview](index.md)
- [Deal and Take Damage (how-to)](../../../how-to/deal-damage.md) — the full recipe.
- [Trigger Hurt](../reference/triggerhurt.md), [Radius Damage](../reference/radiusdamage.md), [Fire Damage](../reference/firedamage.md) — built-in damage-source components.
