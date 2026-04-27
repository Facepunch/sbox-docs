---
title: "Getting Started"
icon: "🚀"
created: 2025-06-15
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Scene/Components/Component.cs
---

# Getting Started

This is the on-ramp. Install the editor, make a project, get something on screen, write your first script. Most of the pages in this section are short.

If you want the bigger picture before you start, read [Architectural Overview](../architecture.md). Otherwise just keep going.

## What you'll write

```csharp
using Sandbox;

public sealed class WelcomeComponent : Component
{
    protected override void OnStart()
    {
        Log.Info( "Welcome to s&box. This runs once when the object spawns." );
    }

    protected override void OnUpdate()
    {
        GameObject.LocalRotation *= Rotation.FromAxis( Vector3.Up, 10f * Time.Delta );
    }
}
```

This is a real component. You attach it to a GameObject in the editor, and the engine handles the rest — calling `OnStart` once, `OnUpdate` every frame, hotloading your edits without restarting.

## The pages in this section, in order

You can take them in order or jump straight to whichever you need.

| Step | Page | What you'll do |
|---|---|---|
| 1 | [Download & Install](installation.md) | Pull the editor down through Steam |
| 2 | [System Requirements](system-requirements.md) | Confirm your machine can run it |
| 3 | [First Steps](first-steps/index.md) | The core concepts — scenes, GameObjects, components, hotload |
| 4 | [A Tour of the Editor](editor-tour.md) | What you see at first launch — panels, toolbars, menus, viewport navigation |
| 5 | [Your First Project](first-project.md) | Make a project, place an object, attach a component, see it move |
| 6 | [Project Types](project-types/index.md) | Game project vs addon — pick the right one for what you're building |
| 7 | [Explore the Engine](explore-engine.md) | The Sandbox Testbed and other learning resources |

When you've got something working, the [Tutorials](../tutorials/index.md) section walks through six guided builds: third-person controller, NavMesh AI, networking lobby, UI, and more.

## Reference info, when you need it

- [FAQ](faq.md) — quick answers
- [Reporting Issues](reporting-errors.md) — where bugs go
- [Monetization](monetization.md) — Play Fund and revenue
- [Feature Status](status.md) — what's done, what's missing

## Things that surprise people

**There's no built-in code editor.** s&box doesn't ship with a text editor for `.cs` files — you write code in Visual Studio, Rider, or VS Code, then save. The engine watches the disk and recompiles on save.

**Hotload pauses on errors.** A red error in the editor console means hotload is paused entirely until you fix it. Save → see error → fix → save again.

**`OnUpdate` runs every frame.** Multiply continuous changes by `Time.Delta` so they're framerate-independent. Don't allocate inside it; cache references in `OnStart`.

**Disabled GameObjects don't tick.** Setting `GameObject.Enabled = false` halts `OnUpdate` for everything underneath it — useful for optimization (disable distant objects) and a common debugging technique.

## Community

If something's stuck and the docs don't help: ask on [the forums](https://sbox.game/f) or in [Discord](https://discord.gg/sbox). Bug reports go in the [sbox-public repo](https://github.com/Facepunch/sbox-public).
