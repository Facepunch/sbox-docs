---
title: "Addon Project"
icon: "🧩"
sources:
  - engine/Sandbox.Engine/Systems/Project/Project/Project.cs
created: 2026-04-27
updated: 2026-04-27
---

# Addon Project

An Addon Project is a content-only package (like a map, 3D model, or material) built specifically to extend an existing Game Project.

## Quick Working Example

You do not write C# code in an Addon Project. Instead, if you need custom logic, you use ActionGraphs.

```csharp
// ❌ C# code is not allowed in Addon Projects.
// Use ActionGraph nodes in the Editor instead!
```

### Selecting a Target Game

When you create or edit an Addon Project, you must define its `Target Game` in the Project Settings. This determines which assets and scripts you are allowed to reference.

![](./images/game-target.png)

:::warning Restart Required
If you change the target game in the project settings, you must restart the s&box Editor for the changes to fully apply and mount the new game's assets.
:::

### Creating Content

You can create any standard s&box asset inside an addon:
*   Maps (`.vmap` / `.scene`)
*   3D Models (`.vmdl`)
*   Materials (`.vmat`)
*   Sounds (`.vsnd`)
*   Custom resources defined by the target game.

### Publishing Addon Assets

Unlike Game Projects which are published as a whole, Addon Projects exist primarily to help you author individual assets. 

To publish something made in an Addon Project (like a custom map), you locate that specific asset in the Asset Browser, right-click it, and select **Publish**. This uploads the individual asset to `sbox.game`, giving it its own dedicated webpage and configuration options.

![](./images/publishing.png)

Since Addon Projects are fundamentally just asset containers, they do not have a dedicated `OnUpdate` loop to optimize. However, if your addon contains an overly complex ActionGraph, be aware that ActionGraphs run through an interpreter and are generally slower than compiled C#. Do not use ActionGraphs for heavy math or per-frame iterations.

## Troubleshooting

:::danger "Compile Error: C# is not allowed in this project type"
If you try to create a `.cs` script inside an Addon Project, the compiler will reject it. You must use ActionGraphs for any logic, or move the logic into the core Game Project if you are the developer of the target game.
:::

:::warning "Missing Assets in Target Game"
If you open an Addon Project and suddenly see pink checkerboards or missing prefabs, verify that the Target Game hasn't updated and removed those assets, and ensure your `.sbproj` is still correctly targeting the right game identifier.
:::

## Related Pages
* [Project Types Index](index.md)
* [Game Project](game-project.md)
* [Publishing Addons](../../publishing/publishing-addons.md)
* [ActionGraph Overview](../../systems/actiongraph/index.md)
