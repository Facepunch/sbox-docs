---
title: Minimal Addon Template
sources:
  - game/templates/addon.minimal
created: 2026-04-27
updated: 2026-04-27
---

# Minimal Addon Template

The **Minimal Addon** template (`game/templates/addon.minimal/`) provides the barebones structure for creating a package that is meant to be consumed by *other* games, rather than played on its own.

## Quick Working Example

If you are building an addon to provide a reusable jump-pad mechanic to the community, you might create a structure like this:

```csharp
// Code/JumpPad.cs (Inside your addon directory)
using Sandbox;

public sealed class JumpPad : Component, Component.ITriggerListener
{
    [Property] public float JumpForce { get; set; } = 500f;

    public void OnTriggerEnter( Collider other )
    {
        if ( other.GameObject.Components.TryGet<Rigidbody>( out var rb ) )
        {
            rb.ApplyAbsoluteImpulse( Vector3.Up * JumpForce );
        }
    }

    public void OnTriggerExit( Collider other ) {}
}
```

## Addon vs Game

It is critical to understand the distinction in the project's `.sbproj` configuration:

| Feature | Game Project | Addon Project |
| :--- | :--- | :--- |
| **Playable from Menu** | Yes | No |
| **Has Startup Scene** | Yes | No |
| **Mounts to other Projects** | No | Yes |
| **Contains Code/Assets** | Yes | Yes |

If you try to launch an Addon directly from the Main Menu, it will fail because the engine has no entry point (no startup scene).

1. **Asset Packs:** You can create an addon that contains zero code, serving solely to distribute a themed pack of 3D models and textures (e.g., "Sci-Fi Corridors Asset Pack").
2. **Systems & Libraries:** You can distribute complex gameplay systems (e.g., an Inventory System, a Vehicle Physics Controller) via an addon so multiple developers can utilize them without copy-pasting code.
3. **Prefabs:** The most powerful way to distribute an addon is by providing ready-to-use Prefabs. If you make a cool enemy AI, bundle the mesh, material, and C# script into a single `.prefab` file inside your addon. Developers can then just drag and drop your Prefab into their scenes.

## Walkthrough: Creating an Addon

1. Open the s&box Editor and click **New Project**.
2. Select **Minimal Addon** and choose a directory.
3. Open the project. Notice that there is no default `.scene` file, only an `Assets` folder.
4. Create a new `Code` folder and add your reusable C# components.
5. Create prefabs that utilize your components.

### Adding Code to Your Addon

While the template starts without a `Code` directory, adding one is standard practice. When you use the Editor to create a new Component, the engine utilizes standard template files (like `default.cs` and `component.cs` located in `game/templates/`) to generate the initial file structure for you. These files will be compiled and distributed as part of your addon.

Any components you create here will be exposed to the parent game that mounts your addon, meaning game developers can attach your components to their GameObjects.
6. When you are ready to share it, you can upload it to the s&box backend, and other developers can add it to their games via the project settings dialog.

- **Namespace Collisions:** When building an addon, it is highly recommended to place your code inside a unique namespace (e.g., `namespace MyName.CoolInventory;`). If you define a class named `PlayerController` in the global namespace, and the host game also defines a `PlayerController`, the compiler will throw ambiguous reference errors and break the user's game.

## Troubleshooting

:::warning Common Gotchas
- **"I can't play my addon!"**
  Addons cannot be played directly. To test an addon, you must create a separate **Game** project, open its Project Settings, and add your local Addon project as a dependency. Then, test your addon's logic within the Game's scenes.
- **Assets are missing when others use my addon:**
  Ensure you are not referencing materials or models that only exist on your local hard drive outside of the addon directory. All required assets must be inside the addon's folder hierarchy.
:::

## Related Pages
- [Publishing Addons](../../publishing/publishing-addons.md)
- [Minimal Library Template](library-minimal.md)
