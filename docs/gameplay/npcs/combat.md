---
title: "Combat & Damage"
icon: "⚔️"
created: 2026-11-20
updated: 2026-11-20
---

# Combat & Damage

`BaseCombatNpc` builds on `BaseNpc` with weapon combat. It finds hostile targets, moves into range, fires in bursts, reloads and searches for enemies it loses sight of.

Set it up with the renderer, animator, movement and sensing layers described in [NPCs](/gameplay/npcs/index.md). Its faction (`Enemy` by default) and relationships decide who it fights.

# Weapons

Assign a child weapon to **Weapon**, or add prefabs to **Weapon Choices** to pick one at random when it spawns. `ArmWith( weaponPrefab )` replaces the weapon at runtime on the host.

Weapons use the same `BaseCombatWeapon` and inventory API as player weapons. The NPC uses the weapon's **NPC** settings for engagement range, burst length, pauses and spread. Games can use the weapon's `UsableByNpcs` flag to filter their weapon choices.

![Different NPC models use the same weapon and inventory API. 1920x1080](./images/weapons.mp4)

The NPC adds a `BaseNpcInventory` if it needs one and doesn't already have an inventory. This has an unlimited reserve, but magazines still empty and need reloading. Add your own inventory component if you want different ammo rules.

**Idle Behaviour** controls whether it holds its post or patrols around where it spawned. Set `Leader` on the host to have it follow another GameObject between fights.

# Damage

`BaseNpc` implements `Component.IDamageable`. Deal damage on the host through `Component.IDamageable.OnDamage()`, like any other [damageable](/scene/components/component-interfaces/index.md). Setting `Health` directly is useful for healing, but doesn't produce a hit.

Override `OnDamaging()` to change incoming damage, or `OnDamaged()` to react after it is applied:

```csharp
protected override void OnDamaging( Component.IDamageable.DamageEvent hit )
{
    if ( hit.DamageInfo.Tags?.Has( "bullet" ) == true )
        hit.AcceptedDamage *= 0.5f;
}
```

`DamageInfo` is the original hit. `AcceptedDamage` is the amount the NPC takes, and `Instigator` is who is responsible for it. Set `Cancelled` during `OnDamaging` to reject a hit.

![An NPC changes incoming damage and reacts to death. 1920x1080](./images/damage.mp4)

# Death and game rules

Override `OnDeath()` for an NPC's death behaviour. Call the base implementation to keep its usual ragdoll and death stimulus. Override `CreateRagdoll()` if you only need to change the corpse.

For rules that apply across the scene, implement `Component.IDamageable.IEvents` on a component or scene system. NPCs and Sandbox players raise `OnDamaging`, `OnDamaged`, `OnDying` and `OnDied` on the host.

`OnDying` can cancel death or set `CreateRagdoll = false`. `OnDied` reports the final instigator and ragdoll before the victim is destroyed. Other `IDamageable` implementations must raise these events themselves.
