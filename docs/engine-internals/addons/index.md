---
title: Addons & Extensions
icon: "🧩"
sources:
  - game/addons
  - engine/Sandbox.Engine/Systems/Project/ProjectConfig.cs
created: 2026-04-27
updated: 2026-04-27
---

# Addons & Extensions

In s&box, **Addons** are the primary way to create, distribute, and consume content. Almost everything you make—whether it's a full game, a map, a collection of models, or a UI library—is packaged as an addon project.

## Quick Start
```csharp
// Example: Accessing a file from an active addon via the virtual filesystem
if ( FileSystem.Mounted.FileExists( "models/my_cool_addon/weapon.vmdl" ) )
{
    Log.Info( "Found the addon weapon!" );
}
```

## Common Patterns

1. **Dependency Mounting:** Your addon can specify other addons it requires in its `.sbproj`. The engine will ensure those dependencies are downloaded and mounted into the virtual filesystem before your code executes.
2. **Asset Sharing:** If you want to share a cool player model you made, you package it as a library addon and publish it. Other developers can then declare it as a dependency in their projects to use your model.
3. **Core Inheritance:** Most standalone games will depend on the `base` addon to inherit fundamental engine classes, networking components, and standard UI features.

## Standard Engine Addons

The engine comes with several built-in addons located in the `game/addons/` directory. These provide foundational content and features that your custom addons can build upon:

*   **[Base Addon](base.md):** The core framework and base classes for creating games. Most games will depend on this.
*   **[Citizen Addon](../../build-games/addons/citizen/index.md):** Contains the default "Citizen" player model, animations, and related assets.
*   **[Menu Addon](index.md):** The main menu interface for the s&box client.
*   **[Tools Addon](tools.md):** Assets and features used primarily by the s&box development editor.

## Troubleshooting

:::warning Common Gotchas
- **Addon not loading / "Missing Dependency":** Ensure the dependencies listed in your `.sbproj` are exactly spelled correctly (e.g., `facepunch.citizen`). If the identifier is wrong, the engine cannot download it.
- **Conflicts between addons:** If two addons try to override the same virtual filesystem path (e.g., both provide a file at `materials/wall.vmat`), the engine will pick one based on load order. It is best practice to nest your assets in a folder named after your addon (e.g., `materials/myaddon/wall.vmat`).
- **Code Namespace Collisions:** Always place your C# classes inside a `namespace` unique to your project to avoid conflicts with other mounted addons.
:::

- **Editor-Only Addons:** You can create addons that only provide tools for the s&box editor. These addons will not be downloaded by players when they join a server.

## Related Pages
- [Runtime Mounting](../runtime-layout/mounting.md)
- [Publishing Addons](../publishing/publishing-addons.md)
- [Minimal Addon Template](../build-games/samples-templates/addon-minimal.md)

