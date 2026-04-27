---
title: "Your First Project"
icon: "🚀"
sources:
  - engine/Sandbox.Engine/Scene/Components/Component.cs
updated: 2026-04-25
created: 2026-04-27
---

# Your First Project

A spinning cube in five minutes. By the end you'll have written one C# component, attached it to a GameObject in the scene, hit Play, and seen your changes hotload while the game runs. The whole loop on which everything else builds.

If you haven't read [First Steps](first-steps/index.md) yet, do that first — it covers the core concepts in 30 seconds. If the editor UI itself looks unfamiliar, [A Tour of the Editor](editor-tour.md) names every panel referenced below.

## The whole component, up front

Here's the code we'll write. Nothing else in this tutorial is more important than understanding this:

```csharp
using Sandbox;

public sealed class SpinComponent : Component
{
    [Property] public float SpinSpeed { get; set; } = 90f;

    protected override void OnUpdate()
    {
        GameObject.WorldRotation *= Rotation.FromAxis( Vector3.Up, SpinSpeed * Time.Delta );
    }
}
```

> **Pause and predict.** Before you read on, try answering these from the code alone:
>
> 1. How often does the line inside `OnUpdate` run?
> 2. If you removed `* Time.Delta`, what would happen on a fast computer vs. a slow computer?
> 3. Where does the value `90f` end up — in the source code? in the editor UI? both?
>
> <details><summary>Answers</summary>
>
> 1. **Every rendered frame.** That's what overriding `OnUpdate` on a `Component` opts you into.
> 2. The cube would **spin faster on faster machines** — at 144 FPS the rotation accumulates 144 increments per second, at 30 FPS only 30. `Time.Delta` (seconds since last frame) is what cancels that out: 144 small increments × short delta == 30 large increments × long delta.
> 3. **Both.** The default lives in the source as `= 90f`, and the `[Property]` attribute exposes it to the Inspector — meaning a designer can override it visually without touching the source file. The default is what shows up the first time the component is added.
>
> </details>

If that already makes sense, skip to Step 4 and just do it. The rest of the tutorial is for if any line raised a question.

## Step 1: New project

Launch `sbox-dev.exe`. The Welcome screen offers a few project types.

1. Click **New Project**.
2. Pick **Game Project**.
3. Choose a folder. Anywhere works — local SSD is best for hotload speed.
4. Name it `MyFirstGame` (or whatever).
5. Click **Create**.

The editor opens with an empty scene. The left panel is the **Scene Tree** (your GameObjects); the right is the **Inspector** (the selected object's components); the bottom is the **Assets Browser** (your project files); the middle is the 3D viewport.

## Step 2: Write the component

Components are `.cs` files. You write them in an external IDE (Visual Studio, Rider, VS Code) — s&box doesn't ship with one.

1. In the Assets Browser, right-click → **Create → C# Component** → name it `SpinComponent`.
2. The editor doesn't open the file for you. Open `SpinComponent.cs` in your IDE — it's in your project's `Code/` folder, or wherever you created it.
3. Replace its contents with the code at the top of this page.
4. **Save the file.** The s&box editor watches the disk and recompiles on save. Look at the editor's console (bottom of the window) — you should see a successful compile within a second or two. If you see red errors, fix them before continuing.

## Step 3: Build the scene

We need something on screen to spin.

1. Right-click in the Scene Tree → **Create Empty GameObject**. Name it `Spinning Box` if you want.
2. With it selected, look at the Inspector. Click **Add Component** → search "Model Renderer" → add it.
3. In the new ModelRenderer's properties, set **Model** to `models/dev/box.vmdl`. (`dev/` are the engine's debug models; every project has them.) A box appears in the viewport.
4. Click **Add Component** again. Search "SpinComponent" — your component should appear in the list. Add it.
5. The SpinComponent has one property: `SpinSpeed`, defaulted to 90.

Save the scene now: **File → Save Scene As** → `main.scene` in your `Assets/` folder. (Habit: save scenes early. Hotload preserves running state but not unsaved scenes.)

## Step 4: Play

1. Click the green **Play** button in the top toolbar (or press `F5`).
2. Your box rotates. Drag the SpinSpeed slider in the Inspector while it runs — the rotation speeds up or slows down live. That's hotload + property reflection working together.
3. Press `F5` again (or click Stop) to exit play mode.

You just shipped a complete s&box loop: write code → save → hotload compiles it → attach to a GameObject → press Play → see it work.

> **Pause and predict.** Stop the game. In your IDE, change the `Vector3.Up` in your component to `Vector3.Forward`. Save (don't restart the editor). What do you expect to happen the next time you press Play?
>
> <details><summary>Answer</summary>
>
> The cube spins around the **forward axis** instead of the up axis. More importantly: the editor doesn't restart, the scene doesn't reload — when you press Play, the new behaviour is already live. That's hotload working between play sessions. Save → Play, no rebuild.
>
> </details>

## What each line of the component is doing

```csharp
[Property] public float SpinSpeed { get; set; } = 90f;
```
The `[Property]` attribute tells the editor to surface this field in the Inspector. Without it, the field exists but is invisible to designers. The `= 90f` is the default — appears as the initial value when you add the component.

```csharp
protected override void OnUpdate()
```
`OnUpdate` is called by the engine every rendered frame. Override it on a `Component` and the engine starts ticking your code. There are other lifecycle methods (`OnAwake`, `OnStart`, `OnFixedUpdate`, `OnDestroy`) — see [Component Lifecycle](../scene/components/component-lifecycle.md).

```csharp
GameObject.WorldRotation *= Rotation.FromAxis( Vector3.Up, SpinSpeed * Time.Delta );
```
- `GameObject.WorldRotation` is the rotation of the GameObject this component is attached to, in world space.
- `Rotation.FromAxis( axis, degrees )` builds a rotation around an axis.
- `SpinSpeed * Time.Delta` is "this many degrees per frame, scaled by how long the frame took." Multiplying by `Time.Delta` is what makes the speed framerate-independent — at 144 FPS the rotation per frame is smaller, but the rotation per *second* is the same.
- `*=` composes rotations. The new rotation is the old rotation followed by the small per-frame increment.

## Things that go wrong

**Object isn't spinning.**
- Did you click Play? The scene only ticks while playing.
- Is the GameObject Enabled? Check the box next to its name in the Inspector.
- Is the SpinComponent attached and Enabled?
- Is `SpinSpeed` set to 0? If so, no rotation per frame.
- Is the box symmetrical from above? A sphere spinning around its up-axis looks identical to a non-spinning sphere. Use a box.

**SpinComponent doesn't show up in Add Component menu.**
- Did you actually save the `.cs` file? Watch the editor console for compile output.
- Is the class `public`? Internal/private classes don't appear in the Add Component menu.
- Are there compile errors elsewhere in the project? Hotload pauses entirely on errors.

**Red errors in the editor console.**
- C# is case-sensitive: `OnUpdate`, not `Onupdate` or `onUpdate`.
- Did you forget the `using Sandbox;` at the top?
- Did the IDE auto-import a wrong namespace? Check that `Component`, `Rotation`, `Vector3`, `Time` all resolve to the `Sandbox.*` namespace.

## Try it yourself

You've got the loop in your hands. Before moving on, try one of these — without copy-pasting:

1. **Make the cube also bob up and down** while it spins. Hint: `GameObject.LocalPosition`, `MathF.Sin( Time.Now )`, and a new `[Property] public float BobAmount { get; set; }`.
2. **Make a second component** called `ColorCycle` that pulses a `ModelRenderer`'s tint between two colours. Hint: `Components.Get<ModelRenderer>()` to find the renderer, then write to its `Tint` property.
3. **Stop the spin** when the player presses Space. Hint: `if ( Input.Down("jump") )` inside `OnUpdate`.

<details><summary>If you get stuck on any of these</summary>

These all use the same loop — write a Component, save, hotload, see effect — but introduce new APIs. The pages that cover them:

- Bob up and down → [Math Types](../code/code-basics/math-types.md) for `Vector3`/`MathF`/`Time.Now`.
- Get another component → [Component lookup patterns](../scene/components/index.md#getting-components).
- React to input → [Detect Player Input](../how-to/detect-input.md).

Don't worry about getting them right the first time. The point is to feel the cycle: type, save, look at the editor, see whether it worked, fix.

</details>

## Where to go next

- **Understand the lifecycle properly** → [Your First Component](../tutorials/your-first-component.md), the line-by-line walkthrough including `OnAwake`, `OnStart`, lifecycle hooks, `[Range]`/`[Group]`/`[ShowIf]`.
- **Build a real character** → [Build a Third-Person Controller](../tutorials/build-a-third-person-controller.md).
- **See what a complete game looks like** → [Sweeper Sample](../build-games/samples-templates/sweeper.md).
- **Already done these and want a curriculum** → [Learning Path: Game Developer](../learning-path-game-dev.md).
