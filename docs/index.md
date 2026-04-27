---
title: "S&box Documentation"
icon: "🍌"
created: 2023-10-26
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Scene/Components/Component.cs
---

# S&box Documentation

What are you trying to do?

## Just playing games

You don't need any of the developer documentation. The whole [For Players](for-players/index.md) section covers installing s&box, finding games, customizing your avatar, and reporting issues. The rest of this site is for people building games or working on the engine.

## Building a game with s&box

You write game code as `Component` classes in C# (small modular pieces of behaviour) and attach them to `GameObject`s (the things in your scene). Hot reload (your code recompiles and live-swaps when you save), scene editor, multiplayer, the works.

- **Brand new** → [Your First Project](getting-started/first-project.md). Spin up a project, get an object moving, see hotload work.
- **Want a curriculum** → [Learning Path: Game Developer](learning-path-game-dev.md). Pages to read in order from "I just installed s&box" to shipping.
- **Already coding, want a specific recipe** → [Cookbook & How-To](how-to/index.md). 30+ recipes by goal.
- **Want to understand the architecture first** → [Architectural Overview](architecture.md).

For the full developer track index see [Build Games](build-games/index.md).

## Looking something up

You're not learning, you're searching for an answer.

- **Term I don't recognize** → [Glossary](glossary.md). ~150 entries; if it has a name in s&box, it's there.
- **"How do I...?"** → [Cookbook](how-to/index.md).
- **Something is broken** → [Troubleshooting](troubleshooting/index.md). 230+ symptoms organized by what you're seeing.
- **API for a built-in component** → [Component Reference](scene/components/reference/index.md).
- **A specific subsystem** → [Engine Systems](systems/index.md) (physics, networking, audio, UI, etc.).
- **A keyboard shortcut or editor feature** → [Editor](editor/index.md).

## I'm coming from another engine

s&box uses its own vocabulary. Quick translation:

| You called it | s&box calls it |
|---|---|
| Actor (Unreal) / GameObject (Unity) | `GameObject` |
| ActorComponent (Unreal) / Component (Unity) | `Component` |
| Script | `Component` (there's no separate script type) |
| Level / Map | `Map` (loadable level asset) or `Scene` (the runtime tree) |
| Blueprint (Unreal) | `ActionGraph` (visual scripting) — though most s&box devs just write C# |
| Prefab | `Prefab` (same name) |
| AudioSource | `SoundPointComponent` |
| Rigidbody | `Rigidbody` (same name) |
| Material | `Material` (same name; `.vmat` extension) |
| Multicast / NetworkVariable | `[Rpc.Broadcast]` / `[Sync]` |

Full mapping plus dozens more terms in the [Glossary](glossary.md).

If you're coming from **Unity**, the model is closest. GameObjects and Components map almost 1:1.

If you're coming from **Unreal**, you'll find the Component model lighter than ActorComponents — there's no Actor base class to inherit from, just GameObjects and Components. Replication uses attributes (`[Sync]`/`[Rpc.*]`) rather than `UPROPERTY`/`UFUNCTION` reflection macros.

## What is s&box

s&box is a game engine built on **Source 2** (the native engine that powers Half-Life: Alyx, originally written by Valve in C++) with **C#** at the gameplay layer. Source 2 handles the heavy stuff — rendering, physics, audio, networking transport. C# sits on top — the **Scene System** (a runtime tree of GameObjects), the editor (`sbox-dev.exe`), **hotloading** (live code reload without restart), and the public API every game dev consumes. You write `.cs` files; the engine compiles and hot-reloads them in milliseconds.

If that sentence raised more questions than answers, [Architectural Overview](architecture.md) is the prose version.

## Quick code example

If you just want to see what s&box C# looks like:

```csharp
using Sandbox;

public sealed class SimpleMover : Component
{
    [Property] public float Speed { get; set; } = 100f;

    protected override void OnUpdate()
    {
        GameObject.WorldPosition += GameObject.WorldTransform.Forward * Speed * Time.Delta;
    }
}
```

That's a real working component. Attach it to a GameObject in the editor and it moves forward at 100 units/second. Save the file and the engine recompiles and hotloads. No restart needed.

## Community

- [Forums](https://sbox.game/f) — community Q&A and discussion
- [Discord](https://discord.gg/sbox) — `#beginners` channel for help
- [GitHub: sbox-public](https://github.com/Facepunch/sbox-public) — bug reports, feature proposals, PRs

For reporting an error from inside the engine, [Reporting Errors](getting-started/reporting-errors.md) covers the workflow.
