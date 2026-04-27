---
title: "Publishing Addons"
icon: "🧩"
created: 2024-10-03
updated: 2026-04-12
sources:
  - docs/getting-started/project-types/addon-project.md
---

# Publishing Addons

An addon project creates content that adds to an existing Game Project, such as maps, models, materials, or custom resources. 

Unlike a standalone game or a primary game package, an addon project itself isn't published directly. Instead, you create assets inside the addon project and publish those assets individually.

:::info
Currently, you cannot make addon projects that contain C# code. You can, however, use ActionGraph for logic.
:::

## Game Target

Because an addon targets a specific game, it needs to know what game you are making content for. This allows your addon to access and use the components, materials, and other assets defined by that target game.

In your Addon Project settings, you can select the target game.

![](../getting-started/project-types/images/game-target.png)

:::warning
If you change the target game in your project settings, you must restart the Editor for the changes to apply.
:::

## Publishing Assets

To publish something you've made in your Addon Project (like a custom map or a 3D model), you do so directly through the Asset Browser.

1. Open the **Asset Browser** in the Editor.
2. Find the asset you want to publish (e.g., your `.vmap` or `.vmdl` file).
3. Right-click the asset and select **Publish**.

![](../getting-started/project-types/images/publishing.png)

From there, your asset will receive its own dedicated page on sbox.game, and you will be able to configure its store presence and details.