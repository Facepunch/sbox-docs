---
title: Walker Map Template
sources:
  - game/templates/walker.map
updated: 2026-04-25
created: 2026-04-27
---

# Walker Map Template

The **Walker Map** template (`game/templates/walker.map/`) provides a straightforward starting point for building a standalone map or simple physics scene that doesn't require a full custom C# game loop.

1. **Map Making:** If you simply want to design a level using Hammer (the mesh editor) or place props in the Scene Editor without touching C# code, this is the template to use.
2. **Physics Testing:** This template is ideal for quickly throwing down rigidbodies and testing physics interactions, constraints, and joint setups.
3. **Showcase Rooms:** Use this template to build visually stunning environments to showcase assets or lighting setups where the player just needs to walk around and look.

## Edge Cases & Limitations

- **No Custom Code Provided:** Out of the box, there is no place to put custom C# logic. If you decide you want to add custom game rules or AI later, you will need to manually create a `Code` directory and ensure your `.sbproj` is configured to compile it.
- **Dependency on Base:** Because it doesn't provide its own player controller script, the scene relies on prefabs and components that exist elsewhere (typically in `game/addons/base/` or `game/addons/citizen/`).

## Troubleshooting

:::warning Common Gotchas
- **"I can't move my character":** If you modified the default scene and removed the player prefab or camera, you will be stuck as a free-cam or stare into the void. Ensure your scene contains a GameObject with a `PlayerController` or similar script if you want walking functionality.

- **Where do I put my scripts?** If you create a script in the root directory and get compiler errors, it's because there's no `Code` folder. Create a `Code` folder manually. Once created, using the Editor to generate a new C# Component will use the standard `default.cs` or `component.cs` templates from `game/templates/` to get you started.
:::

## Related Pages
- [Scene System](../../scene/index.md)
- [Minimal Game Template](game-minimal.md)
