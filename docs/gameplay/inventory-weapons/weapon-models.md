---
title: "Weapon Models"
icon: "✋"
created: 2026-07-04
updated: 2026-09-03
---

# Weapon Models

`BaseWeaponModel` marks up a weapon's `GameObject` with the points it fires from. Put it on the model prefab, assign the renderer and the attachment GameObjects, and the weapon holding it finds everything through there.

## View and world models

A weapon has two looks. The view model renders for the holding player in first person, and the world model attaches to the holder's hand bone (`HoldBone`) for everyone else. Assign `ViewModelPrefab` and `WorldModelPrefab` on the weapon, each with a BaseWeaponModel on it.

`HoldType` and `Handedness` on the weapon set how the holder carries it. They drive the "holdtype" parameters on the holder's animgraph, so the citizen stands right with a pistol, a rifle or nothing at all.

## Attachments

`MuzzleGameObject` marks where shots come from and `ShellEjectGameObject` where brass comes out. They're just empty GameObjects placed on the model.

## Effects

Muzzle flash, brass eject and tracers come with default prefabs, so once the attachments are set shots look right with nothing more to do. Swap them per weapon with the `MuzzleEffect`, `EjectBrass` and `TracerEffect` properties, or clear them to have none.

## Animation

The model reacts to the weapon through hooks, and the base ones set animgraph parameters: deploying sets `b_deploy`, attacking sets `b_attack` and plays the effects, reloading sets `b_reload`. Override `OnDeploy`, `OnAttack`, `OnReloadStart` etc in a subclass when your rig wants something else.

There are ready made first person arms and weapons you can use: [First Person Weapons](/assets/ready-to-use-assets/first-person-weapons.md).
