---
title: "Addon Project"
icon: "🧩"
created: 2024-10-03
updated: 2026-11-18
---

# Addon Project

An addon project adds to a Game Project. The addon project isn't published directly, you create assets and publish those individually.

The general idea is that you are able to use the components and assets from a target game to create assets for that game. The asset could be a map, a model, a material or even a custom resource defined by that game.

For example, to make a map for someone else's deathmatch game, you'd create an addon project targeting that game. You get their spawn points, weapon pickups and game rules to build with, and you publish just the map.

Other common uses:

- A model for a game that loads models from addons
- A material pack for an existing game
- A custom resource the game defines, like a card or a recipe
- A scripted tool or entity for a game that loads scripts from addons



:::info
Addon projects can't contain compiled C# code, but they can contain [script files](/code/scripting/script-files.md) (`.csx`) and [ActionGraph](/actiongraph/index.md). A script file can define components, so an addon can add its own behaviour to the target game.

:::

## Game Target

In the project settings you're able to select the target game. If you change this game then you must restart the editor for the changes to apply.


![](./images/game-target.png)

## Publishing

To publish something made in an Addon Project, you would find it in the asset browser and publish it from there.

![](./images/publishing.png)

From there your map, or model, or whatever will get its own page on sbox.game, and you will be able to configure it.
