---
title: "Cloning"
icon: "🐑"
sources:
  - engine/Sandbox.Engine/Scene/GameObject/GameObject.Clone.cs
  - engine/Sandbox.Engine/Scene/Components/Component.Clone.cs
created: 2026-04-27
updated: 2026-04-27
---

# Cloning

Cloning is the process of generating a deep, exact copy of a `GameObject` and all of its components and children.

When you call `GameObject.Clone()`, the engine does not simply copy values over. It performs a structured serialization and instantiation process to ensure the new object behaves identically to the original without sharing underlying memory references.

## The Cloning Process

1. **Serialization:** The engine serializes the state of the original GameObject, including its hierarchy and all attached components, into an internal JSON structure.
2. **Instantiation:** A new GameObject hierarchy is created based on the serialized data.
3. **Reference Mapping:** During cloning, objects often reference each other (e.g., a component referencing another component on the same GameObject via `ComponentReference` or direct C# fields). The engine automatically builds a map connecting original `Guid`s to the newly generated `Guid`s, ensuring that the clone's internal references point to other parts of the *clone*, not the original.
4. **Initialization:** The new components run their `OnAwake` and `OnStart` lifecycle hooks as they are activated.

> **Analogy:** Cloning isn't like taking a photograph. It's like taking the blueprints for a house, making a photocopy of the blueprints, and then building an identical house next door. The two houses look the same, but turning off the lights in one doesn't affect the other.

## Cloning Custom Components

When a GameObject is cloned, the engine automatically handles copying the data for your custom C# `Component` classes. You do not need to (and cannot) override a `Clone()` method manually on a component.

The engine uses one of two methods to copy component data:
* **JSON Serialization (Safe but slower):** The default method where the component's public `[Property]` fields are serialized to JSON and then deserialized onto the new instance.
* **Copy-by-Value (Fast):** If the engine determines the component type is simple (e.g., it only contains value types or implements `ICloneable` safely), it skips JSON serialization and performs a direct memory copy, making spawning much faster.

## Prefab Links

If the original GameObject is an instance of a Prefab, the clone will also retain that Prefab link. This means if you later update the Prefab file on disk, the cloned instances in your world will still receive those updates (assuming they haven't overridden those specific properties).

## Related Pages
- [Clone GameObjects Guide](../how-to/clone-gameobjects.md) (Task-oriented guide)
- [Spawn a Prefab](../how-to/spawn-prefab.md)
