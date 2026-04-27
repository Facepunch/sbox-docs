---
title: "Build Games"
icon: "🔨"
sources:
  - engine/Sandbox.Engine/Scene/Components/Component.cs
updated: 2026-04-25
created: 2026-04-27
---

# Build Games

This is the developer track. Everything you need to make a game or addon in s&box lives under this section. If you're playing games rather than building them, head to [For Players](../for-players/index.md) instead.

## Quick Working Code Example

A typical s&box game is a collection of `Component` scripts attached to `GameObject`s. Here's a component that lets the player jump on a button press:

```csharp
using Sandbox;

public sealed class PlayerJump : Component
{
    [Property] public float JumpForce { get; set; } = 400f;
    private Rigidbody rb;

    protected override void OnStart()
    {
        // Cache the Rigidbody component for performance
        rb = Components.Get<Rigidbody>();
    }

    protected override void OnUpdate()
    {
        if ( Input.Pressed( "jump" ) && rb != null )
        {
            rb.ApplyImpulse( Vector3.Up * JumpForce );
        }
    }
}
```

## Common Patterns

1. **Component-Based Design.** Avoid monolithic managers. Build small, single-purpose components (`PlayerJump`, `Health`, `Inventory`) and combine them on GameObjects.
2. **Editor Properties.** Use the `[Property]` attribute to expose tuning values (speed, health, colors) to the Inspector so designers can tweak without recompiling.
3. **Prefabs.** Build complex objects (an enemy, a weapon) once, save them as a Prefab, and spawn dynamically via `Scene.PrefabSystem.Spawn()`.

## Where to Start

If you're new, work through the track in roughly this order:

0. **[Architectural Overview](../architecture.md)** — how the engine fits together, top to bottom. Worth reading first if you want context; skippable if you just want to start coding.
1. **[Getting Started](../getting-started/index.md)** — Install s&box, create your first project, get an object spinning.
2. **[Tutorials](../tutorials/index.md)** — Six guided walkthroughs covering components, third-person controllers, NavMesh AI, networking, and UI.
3. **[Samples & Templates](samples-templates/index.md)** — Working projects (Sweeper game, Walker map, minimal addon/game/library templates) you can dissect.

Once oriented, dip into:

- **[Cookbook & How-To](../how-to/index.md)** — 30+ goal-oriented recipes (player movement, raycasts, save/load, sync variables, async tasks, …).
- **[Scene System](../scene/index.md)** — GameObject, Component lifecycle, Prefabs, Tags, Transform, the full Component Reference.
- **[Code Basics](../code/index.md)** — Math types, hotloading, the API whitelist, console variables, code generation.
- **[Engine Systems](../systems/index.md)** — Physics, Audio, VR, Navigation, Networking, UI, Shaders, Movie Maker, ActionGraph, Effects, Post-Processing, Services, Platform integration, Performance.
- **[Addons & Extensions](../engine-internals/addons/index.md)** — The Base, Citizen, Menu, and Tools addons that ship with s&box.
- **[Editor](../editor/index.md)** — Hammer (mapping), Shader Graph, Movie Maker, Action Graph, custom editor tools and widgets.
- **[Publishing](../publishing/index.md)** — Exporting standalone, publishing addons.
- **[API Reference](../reference/index.md)** — Lookup index for namespaces and types.

## Related Pages
- [Engine Internals](../engine-internals/index.md) — The other side of the wall: native binding layer, hotload pipeline, AppSystem, native Qt integration. Generally not needed for game development.
- [For Players](../for-players/index.md) — Player-facing track for installing and playing games.
