---
title: "Inventory & Weapons"
icon: "🧰"
created: 2026-07-04
updated: 2026-11-20
---

# Inventory & Weapons

The engine ships with an inventory and weapon system, modelled loosely on Garry's Mod. It gives you pickups, slots, weapon switching, ammo, shooting and reloading without you having to build any of it.

There are three components:

* [BaseInventoryComponent](/gameplay/inventory-weapons/inventory.md) goes on your player or NPC. It holds items, tracks which one is deployed, and owns the reserve ammo pool.
* [BaseInventoryItem](/gameplay/inventory-weapons/inventory.md) is anything that can live in an inventory. Usable as-is for a simple pickup.
* [BaseCombatWeapon](/gameplay/inventory-weapons/weapons.md) builds on that to make guns, melee weapons and tools.

A working gun needs no code at all. Put a BaseCombatWeapon on a prefab, fill in the properties, and it shoots, reloads, runs dry and plays its effects. You subclass when you want it to behave differently.

Everything is networked. The host owns the truth, and the holding player predicts locally so firing feels instant.
