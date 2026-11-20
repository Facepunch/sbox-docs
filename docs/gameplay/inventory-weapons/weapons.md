---
title: "Weapons"
icon: "🔫"
created: 2026-07-04
updated: 2026-11-20
---

# Weapons

`BaseCombatWeapon` is the base for anything a player holds and uses. Guns work straight out of the box, and it stretches to melee weapons and tools with a small subclass. [NPCs](/gameplay/npcs/combat.md) can use the same weapons.

## Setting up a gun

Make a prefab with a BaseCombatWeapon on it and fill in the inspector:

* **Shooting** has the fire rates, whether each trigger is automatic, and the `Ballistics` struct that drives the default attack: damage, pellets, range, spread and spread recovery. One pellet is a rifle, eight is a shotgun.
* **Ammo** and **Reloading** are covered below.
* **ViewModel** and **WorldModel** are the looks. See [Weapon Models](/gameplay/inventory-weapons/weapon-models.md).

That's a working gun. Attack1 shoots, reload reloads, attack2 does nothing until you override it.

## Ammo and clips

A weapon can feed from a magazine (`Clip1`), feed straight from the inventory's reserve pool, or feed from both. `PrimaryAmmoType` picks which pool. Leave it unset for a bottomless reserve, where the magazine still forces the reload rhythm but never runs out.

Secondary fire can have its own magazine and ammo type, for things like grenade launchers.

Turning `UsesAmmo` off takes ammo out of the equation entirely and the weapon never runs dry. That's the setting for melee and tools.

## Reloading

Reloads are whole magazine by default. `IncrementalReloading` loads one round at a time instead, shotgun style. `CanCancelReload` lets firing interrupt a reload part way, and `AutoReload` starts one when the trigger is pulled on an empty magazine.

## Doing your own thing

Override `PrimaryAttack` or `SecondaryAttack` to change what firing does. The base primary spends a round, traces the ballistics and shows the shot on every peer. In your own override, call `ShootEffects()` to get the sound, muzzle flash and attack animation. `Think()` runs every frame while deployed, for charge-ups and spin-downs, and `DryFire()` is the click on empty.

### Melee weapons

A melee weapon is a short range trace instead of a ballistic volley. Turn the Ammo feature off, set **Attack Damage Tags** to `melee`, and override the attack:

```csharp
public class Crowbar : BaseCombatWeapon
{
	public override void PrimaryAttack()
	{
		ShootEffects();

		// short, thick and forgiving
		ShootBullet( distance: 80f, radius: 8f, damage: 25f, force: 500f );
	}
}
```

`PrimaryDelay` is the swing rate. That's the whole weapon.

### Tools

Not everything a player holds hurts people. A camera, a grappling hook and a medkit are all BaseCombatWeapon with Ammo turned off and an attack that does something else. You keep deploying, holstering, view models, switching and input handling for free, so a tool is usually just an override of `PrimaryAttack` that does the thing, the same shape as the crowbar above.

## Networking

Weapons are host authoritative and owner predicted. The holding player fires instantly, their hits travel to the host as claims, and the host applies the damage. Override `OnValidateShotClaim` to reject claims your game considers impossible, that's the anticheat hook.

## HUD

Weapons draw a basic crosshair and nothing else. There's no ammo counter, build one yourself. Everything you need is synced, so it's safe to read from HUD code on any client:

```csharp
if ( Inventory.ActiveItem is BaseCombatWeapon weapon && weapon.UsesAmmo )
{
	var clip = weapon.Clip1;     // rounds in the magazine, -1 when there's no magazine
	var reserve = weapon.Ammo1;  // reserve ammo in the holder's pool
}
```

A weapon with no ammo type has a bottomless reserve, so `Ammo1` comes back as a very large number. Check `PrimaryAmmoType` is set before showing a reserve count.
