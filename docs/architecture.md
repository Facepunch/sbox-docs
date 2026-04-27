---
title: "Architectural Overview"
icon: "🏛️"
updated: 2026-04-27
created: 2026-04-27
---

# Architectural Overview

## The 30-second model

When you write this line in a `Component`:

```csharp
GameObject.WorldPosition += Vector3.Up * Time.Delta;
```

Here is what happens when the game ticks:

1. **You wrote it** — in a `.cs` file in your project, which is automatically compiled and hot-loaded.
2. The engine's main loop ticks and dispatches to the active `Scene`, which walks its tree and invokes your `OnUpdate` method.
3. Your line runs. The setter on **`GameObject.WorldPosition`** stores the new transform in the engine.
4. The engine pushes that transform across the boundary into the underlying C++ Source 2 scene.
5. **Source 2's renderer** picks up the moved scene-object and draws it on your screen.

You only ever touched step 1 and step 3. Everything else is the engine architecture — the layers below your code that exist so the surface you write against is as simple and powerful as possible.

## The Layered Stack

The engine is built in layers, from your game code at the top down to the native renderer at the bottom.

1. **Your game code** — The `.cs` files in your project. This includes your custom components, prefabs, UI panels, and game logic. This layer is fully hot-loadable.
2. **The Scene System** — The `GameObject` and `Component` lifecycle, the scene tree, and the core runtime that your code interacts with directly.
3. **Engine Systems** — High-level systems like Physics, Audio, Networking, UI, and Input. The Scene system delegates work to these systems, and you interact with them via the public API.
4. **Core Services** — The underlying plumbing that makes the engine work: hotloading assemblies, compiling Razor UI, sandboxing, reflection, and serialisation.
5. **Native Interop** — The bridge between the managed C# environment and the underlying C++ engine.
6. **Source 2 (C++)** — The foundational native engine that powers rendering, physics simulation (Rubikon), audio mixing, and networking transport.

The boundary that matters most for game developers is the line between Engine Systems and your game code. The engine handles the heavy lifting below that line.

## Asset and Package Model

Your game is treated as a **package** (an addon or game project). Packages are bundles of code and assets. At runtime, packages are **mounted** into a virtual filesystem so your code accesses files by paths like `models/citizen/citizen.vmdl` regardless of where they physically live.

A typical project directory contains:
- `Code/` — Compiled into your hot-loadable assembly.
- `Assets/` — Mounted at runtime and accessed via the filesystem (models, materials, sounds).
- `Scenes/` and `Prefabs/` — Your saved scene configurations.

### Mount Order

When multiple packages provide assets, they are mounted in a specific order (later entries win on conflict):

1. **Core Engine Assets** (`game/core`) — Foundational materials and models.
2. **Base Addons** (`game/addons/base`) — Foundational components.
3. **Your Project's Assets** — These sit at the top of the stack and can override lower-level files if needed.
4. **Mounted External Games** — External game content if explicitly mounted.

## Where to go next

Now that you have a understanding of the architecture, here are some good starting points:

1. [Your First Project](getting-started/first-project.md) — Get a basic project running.
2. [GameObject](scene/gameobject.md) & [Component Lifecycle](scene/components/component-lifecycle.md) — Dive deeper into the core of the scene system.
3. [Networked Objects](systems/networking-multiplayer/networked-objects.md) — Learn how to synchronize state for multiplayer games.
