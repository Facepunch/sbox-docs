---
title: "Explore the Engine"
icon: "🔍"
created: 2026-04-09
updated: 2026-04-27
sources:
  - engine/Sandbox.Engine/Scene/Components/Component.cs
---

# Explore the Engine

S&box is a scene-based engine. This means everything in your game exists within a **Scene**, built from **GameObjects** and **Components**.

## The Scene System

- **Scenes:** Your game world. Every rendered and updated object belongs to a Scene. Scenes are JSON files on disk.
- **GameObjects:** The entities within a scene. They have a Transform (position, rotation, scale) and can be parented to each other.
- **Components:** Modular behavior attached to GameObjects. One object might have a `ModelRenderer` for visuals, a `BoxCollider` for physics, and a custom `PlayerController` for logic.

## Play the Testbed

The best way to see what the engine can do is the `testbed`.

1. Launch s&box (the game client, not the editor).
2. Search for and launch the **testbed** game.
3. Explore the various scenes — each exhibits a specific engine feature (physics, UI, rendering).
4. You can download the source for these scenes at [Facepunch/sbox-scenestaging](https://github.com/Facepunch/sbox-scenestaging) to see exactly how they are built.

## Tips for Exploration

- **Hotloading:** Changes to code or scenes are applied instantly while the game is running.
- **Scene Switching:** Switching between scenes is extremely fast because they are high-level JSON data, not traditional "compiled" maps.
- **Hierarchies:** Keep GameObject hierarchies flat where possible for better performance.
- **Caching:** In your components, cache references to other GameObjects or Components in `OnStart` rather than searching for them in `OnUpdate`.

## Related Pages

- **[Scenes & GameObjects](../scene/index.md)** — Deep dive into the scene system
- **[Code](../code/index.md)** — C# programming in s&box
- **[Editor](../editor/index.md)** — Learn the editor tools
- **[Systems](../systems/index.md)** — Input, networking, audio, and more
