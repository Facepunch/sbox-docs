---
title: "Project Types"
icon: "📂"
created: 2024-10-03
updated: 2026-04-13
sources:
  - engine/Sandbox.Engine/Systems/Project/ProjectConfig.cs
---

# Project Types

When creating a new project in s&box, you are presented with different project types to match your goals. Currently, these are **Game Projects** and **Addon Projects**.

## Quick Working Example

You can programmatically read the current project's configuration (like its Ident) from within a running game:

```csharp
using Sandbox;

public sealed class ProjectInfo : Component
{
    protected override void OnStart()
    {
        // Example: Reading the project ident at runtime
        Log.Info( $"Running Project: {Game.Ident}" );
    }
}
```

## Quick Overview

- **Game Project:** A standalone experience containing a complete game, with scenes, code, maps, and assets.
- **Addon Project:** A content-only project (like a map, model, or material) designed specifically for an existing target Game Project.

1. **Changing the Project Ident:**
   When setting up your project, you'll see options for "Organization Ident" and "Package Ident".
   - `Org`: The organization that owns the project (e.g., `facepunch`). Use `local` if you are just testing offline.
   - `Ident`: A short, lowercase name with no special characters (e.g., `my_cool_game`).
   Your project's full identifier on the backend becomes `org.ident`.

2. **Targeting Games from Addons:**
   When working within an Addon Project, you must select a Target Game in the project settings. This determines which base codebase you inherit from. This pattern is primarily used by level designers creating custom maps for an existing game.

3. **Managing Dependencies:**
   Most Game Projects will depend on the `base` addon. This is configured in the `.sbproj` file. By declaring dependencies, you ensure the engine automatically downloads and mounts those required addons before your project starts.

## Navigation Hub

* [**Game Project**](game-project.md) - Learn how to set up your primary game environment, scenes, and startups.
* [**Addon Project**](addon-project.md) - Learn how to build and publish custom assets for an existing target game.

## Troubleshooting

:::danger "Package already exists"
When creating a project, if you use an Organization and Ident combination that someone else has already published to `sbox.game`, you will get errors when trying to publish. Always use a unique Ident or keep the Org as `local` until you're ready to secure a real name.
:::

:::warning "Missing Dependencies"
If your code fails to compile or assets are missing, check your `.sbproj` dependencies. If you rely on code from another addon (like `facepunch.citizen`), you must explicitly list it as a dependency so the engine mounts it.
:::

## Performance
- **Addon Mounting:** Mounting too many large addon dependencies can slow down the initial load time of your Game Project. Only depend on addons you actively need.

## Related Pages
* [Getting Started](../index.md)
* [Publishing Addons](../../publishing/publishing-addons.md)
* [Exporting Standalone](../../publishing/exporting-standalone.md)
