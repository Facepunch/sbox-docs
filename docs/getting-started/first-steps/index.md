---
title: "First Steps"
icon: "👣"
sources:
  - engine/Sandbox.Engine/Scene/Components/Component.cs
updated: 2026-04-25
created: 2026-04-27
---

# First Steps

The core concepts you need before opening the editor:

- Your **scene** is a tree of **GameObjects**.
- Each GameObject has a transform (position/rotation/scale) and a list of **Components**.
- All your code lives in `Component` classes you write in C#. Override `OnUpdate` to run something every frame, `OnStart` to set things up once.
- Save your `.cs` file and the engine recompiles and hot-reloads in milliseconds. No restart.

That's the whole shape. Everything else is detail.

## A real component

```csharp
using Sandbox;

public sealed class MyFirstComponent : Component
{
    [Property] public float Speed { get; set; } = 100f;

    protected override void OnStart()
    {
        Log.Info( "The component has started!" );
    }

    protected override void OnUpdate()
    {
        GameObject.WorldPosition += GameObject.WorldRotation.Forward * Speed * Time.Delta;
    }
}
```

Three things to notice:

- `using Sandbox;` — every engine type lives there.
- `[Property]` — exposes `Speed` to the Inspector so you can tweak it visually.
- `Time.Delta` — multiply continuous changes by it so movement is framerate-independent.

> **Pause and predict.** If you delete the `[Property]` line from the component above, what changes? What still works the same?
>
> <details><summary>Answer</summary>
>
> The `Speed` field still exists and the code still uses it (default `100f`), so the component still moves on `OnUpdate`. What changes: `Speed` disappears from the Inspector — there's no UI to tweak it without editing the source file. The attribute is purely an editor-surface signal, not a runtime requirement.
>
> </details>

## What a scene looks like

A typical scene is a small tree of GameObjects, each holding a list of components:

- **Scene** (the root)
  - **Main Camera** — holds `CameraComponent`.
  - **Directional Light** — holds `DirectionalLight`.
  - **Player Character** — holds `PlayerController`, `ModelRenderer`, `SkinnedModelRenderer`, `CapsuleCollider`.

A GameObject can hold any number of components, and any component can be on any GameObject — there's no required pairing.

## What's worth knowing now

These three things will save you hours later:

1. **Hotload pauses on errors.** A red compile error in the editor console blocks every subsequent hotload until you fix it. When changes "stop applying," the console is the first place to look.

2. **`[Property]` references must be assigned in the Inspector.** Adding `[Property] public GameObject Target { get; set; }` to your component creates an empty slot in the Inspector. If you don't drag a target into it, `Target` is null at runtime and your code throws `NullReferenceException` on first use.

3. **`OnUpdate` runs every frame.** At 144 FPS, anything you do here happens 144 times a second. Don't allocate (`new List`), don't query the scene (`Scene.GetAllComponents<T>()`), don't load assets. Cache what you can in `OnStart`.

## Where this differs from older Source modding

If you've worked on Garry's Mod, Half-Life 2 mods, or other Source-1-era projects: **the entity system and Hammer-driven gameplay logic are gone.** s&box uses scenes and components like Unity does. There are no `ent_create`, no `func_logic_*`, no Hammer entity I/O for gameplay. Hammer still exists for level geometry but is being deprecated in favour of [Scene Mapping](../../scene/scene-mapping/index.md).

If you've worked in Unity or Unreal: the Scene System maps almost 1:1 onto Unity's GameObject/Component model. The [vocabulary mapping in the glossary](../../glossary.md#vocabulary-from-other-engines) translates the rest.

## Next

When you're ready to do something concrete:

- [**A Tour of the Editor**](../editor-tour.md) — what you actually see when you launch `sbox-dev.exe`: panels, toolbars, menus, viewport navigation. Skip this if you're comfortable poking around editors; read it first if you want to know what each panel is before clicking.
- [**Your First Project**](../first-project.md) — get a cube spinning. ~5 minutes.
- [**Your First Component**](../../tutorials/your-first-component.md) — line-by-line walkthrough of the lifecycle.
- [**Your First Game**](../../tutorials/your-first-game.md) — build a small complete game from scratch.

If you'd rather read more theory first:

- [Architectural Overview](../../architecture.md) — how the engine fits together
- [Scene System](../../scene/index.md) — the GameObject/Component model in detail
