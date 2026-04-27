---
title: Samples & Templates
sources:
  - game/samples
  - game/templates
created: 2026-04-27
updated: 2026-04-27
---

# Samples & Templates

The best way to learn s&box development is by reading, modifying, and experimenting with existing code. The engine provides several sample projects and starter templates to help you get going.

These are located in the `game/samples/` and `game/templates/` directories respectively.

## Quick Start Example

You can explore sample projects using the Editor or programmatically list them. Here is a generic way you might check what addons are installed via code:

```csharp
using Sandbox;

public sealed class SampleScanner : Component
{
    protected override void OnStart()
    {
        // Example of checking for a specific sample addon mount
        if ( FileSystem.Mounted.DirectoryExists( "samples/sweeper" ) )
        {
            Log.Info( "The Sweeper sample is mounted!" );
        }
    }
}
```

## Common Patterns

1. **Studying Samples:** Copy code from samples into your own project instead of modifying the sample directly, so you don't lose the original reference.
2. **Template Customization:** Start with a template, and then modify the `.sbproj` to fit your specific needs, updating the Ident to make it your own.
3. **Exploring the Scene:** The best way to understand a sample is to open its main `.scene` file and inspect the hierarchy of GameObjects and Components.

## Troubleshooting

:::warning Common Gotchas
- **Sample Not Running:**
  If a sample project does not compile or run out of the box, ensure you are running the latest version of the s&box Editor, as breaking API changes occur often.
- **Missing References:**
  If you copy code from a sample into your project and get a missing reference, ensure you also copy the required models/materials/sounds, or update the script to point to assets in your own project.
- **Accidental Changes:**
  If you edit a built-in template's code directly in the engine's `game/templates/` folder, it may be overwritten when the engine updates.
:::

## Performance

When using templates, be aware that code written in `OnUpdate` or similar hot paths can be costly. For example, if a sample contains a component checking inputs or physics queries every frame, ensure you remove or optimize these when adapting them for heavier game scenarios.
Always measure performance via the in-editor profiling tools (such as SboxProfiler).

- Samples might rely on certain Addons like `facepunch.citizen`. If you create an empty project and copy a sample's components without copying its `.sbproj` dependencies, you'll encounter missing asset errors. Ensure all required Addons are mounted.

## Samples

Samples are complete, working examples of specific concepts or small games. They are designed to show best practices and how different engine systems integrate.

*   **[Sweeper](sweeper.md):** A simple, fully functional Minesweeper clone demonstrating basic UI, input handling, and game state.

## Templates

Templates are bare-bones projects designed to be the starting point for your own creations. When you create a new project in the s&box editor, you choose one of these templates.

*   **[Minimal Addon](addon-minimal.md):** The most basic empty addon structure.
*   **[Minimal Game](game-minimal.md):** A basic game setup with a scene and a simple player controller.
*   **[Minimal Library](library-minimal.md):** A template for creating code-only libraries intended to be shared among other addons.
*   **[Sandbox Addon](sandbox-addon.md):** A template geared towards creating content meant to be spawned in the base Sandbox game mode.
*   **[Walker Map](walker-map.md):** A basic map template demonstrating terrain, lighting, and spawn points.

## How to Use Them

To learn from a sample:
1.  Open the s&box editor.
2.  Open the sample project folder (e.g., `game/samples/sweeper/`).
3.  Press Play to see how it works.
4.  Examine the code and try changing values or adding new features to see the results.
