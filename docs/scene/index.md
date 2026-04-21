---
title: "Scene"
icon: "🪑"
created: 2023-11-14
updated: 2026-04-10
---

# Scene

S&box uses a scene system, similar to Godot and Unity. Scenes are JSON files on disk that are fast to load and switch between.

Everything in your game is built from Scenes, GameObjects, and Components working together.

## [Scenes](/scene/scenes/index.md)

A Scene is your game world. Everything that renders and updates at one time should be in a scene. Scenes can be saved to disk, loaded, and switched between at runtime.

## [GameObjects](/scene/gameobject.md)

A scene contains GameObjects — world objects with a position, rotation, and scale. They can be arranged in a hierarchy so children move relative to their parents.

## [Components](/scene/components/index.md)

GameObjects contain Components that provide modular functionality. A ModelRenderer renders a model, a BoxCollider makes it solid. You create games by programming new Components.

## [GameObjectSystem](/scene/gameobjectsystem.md)

Systems that operate on GameObjects across an entire scene — useful for game managers and global logic.

## [Prefabs](/scene/prefabs/index.md)

Reusable GameObject templates that can be instantiated at runtime or placed in scenes. Supports instance overrides and nested prefabs.
