---
title: "Build Your First Component"
icon: "🚀"
sources:
  - engine/Sandbox.Engine/Scene/Components/Component.cs
  - engine/Sandbox.Engine/Scene/Components/Component.Update.cs
  - engine/Sandbox.System/Attributes/Property.cs
  - engine/Sandbox.System/Attributes/Range.cs
  - engine/Sandbox.System/Attributes/InspectorAttributes.cs
updated: 2026-04-25
created: 2026-04-27
---

# Build Your First Component

The Component is the unit of behavior in s&box. Everything from a player controller to a particle effect to your own custom logic is a Component attached to a GameObject. This walkthrough writes one from scratch, then digs into the lifecycle, properties, and how components find each other.

## What you'll build

A `SpinComponent` that rotates whatever GameObject it's attached to. Then a `SpinTarget` that listens for the spin and emits a sound, so we have an excuse to talk about how components reference each other.

## Prerequisites

You have a project open in the Editor with at least one scene saved. If you don't, the first half of [Your First Game](your-first-game.md) covers project creation.

---

## Step 1: Create the C# file

In the **Assets Browser**, right-click and pick **Create -> C# Component**. Name it `SpinComponent.cs` and double-click to open it in your editor (Visual Studio, Rider, or VS Code — s&box generates the project file each time it compiles).

> If "Create -> C# Component" isn't in the menu, you're probably right-clicking inside an asset (like a `.scene` file) instead of an empty area of the browser. Click an empty folder first.

The default template is essentially the snippet below — replace whatever it generated:

```csharp
using Sandbox;

public sealed class SpinComponent : Component
{
    [Property] public float SpinSpeed { get; set; } = 90f;

    protected override void OnUpdate()
    {
        // Rotate around the world Z axis at SpinSpeed degrees per second.
        GameObject.WorldRotation *= Rotation.FromAxis( Vector3.Up, SpinSpeed * Time.Delta );
    }
}
```

Save. The Editor watches your code folder, recompiles automatically, and hot-loads the new type into the running game (no restart needed). Watch the bottom-right of the Editor — you'll see "Compiling" and then a green check.

> **If compilation fails**, the Editor pops a console panel listing the error and line number. Hot-load won't apply broken code, so the previous version stays running until you fix it.

### Checkpoint

Switch back to the Editor. With a GameObject selected (any one will do), click **Add Component** and type `Spin`. `SpinComponent` should appear. If it doesn't, your code didn't compile — open the **Console** (`View -> Console`) and look for red text.

---

## Step 2: Set up a visible target

We need something to actually see spinning.

1. Right-click the **Scene Tree** and pick **Create Object -> 3D Object -> Cube**. This creates a GameObject with `ModelRenderer` and `BoxCollider` already attached.
2. Select the new Cube, then in the Inspector click **Add Component**.
3. Search `SpinComponent` and add it. The `SpinSpeed` property shows up in the Inspector with its default `90`.

> If the Cube uses `models/dev/box.vmdl` you'll see a checkerboard. The `models/dev/` folder ships with the `facepunch.base` addon — if your project doesn't reference it, you'll get a missing-asset placeholder instead. Open `your-project.sbproj` and confirm `facepunch.base` is in the `Mounts` list.

Press **Play**. The cube spins counter-clockwise around its vertical axis. While running, drag the `SpinSpeed` slider in the Inspector — the rotation rate updates immediately. That's because `OnUpdate` reads `SpinSpeed` every frame, and the Inspector writes directly to the live property.

---

## Step 3: Understand the lifecycle

Stop the game. Open `SpinComponent.cs` and add the rest of the lifecycle hooks so you can watch them fire:

```csharp
using Sandbox;

public sealed class SpinComponent : Component
{
    [Property, Range( 0, 720 )] public float SpinSpeed { get; set; } = 90f;

    protected override void OnAwake()
    {
        Log.Info( $"OnAwake on {GameObject.Name}" );
    }

    protected override void OnStart()
    {
        Log.Info( $"OnStart on {GameObject.Name}" );
    }

    protected override void OnEnabled()
    {
        Log.Info( $"OnEnabled on {GameObject.Name}" );
    }

    protected override void OnUpdate()
    {
        GameObject.WorldRotation *= Rotation.FromAxis( Vector3.Up, SpinSpeed * Time.Delta );
    }

    protected override void OnDisabled()
    {
        Log.Info( $"OnDisabled on {GameObject.Name}" );
    }

    protected override void OnDestroy()
    {
        Log.Info( $"OnDestroy on {GameObject.Name}" );
    }
}
```

> **Pause and predict:** Before you press Play, what order will those three logs (`OnAwake`, `OnEnabled`, `OnStart`) print in — and why? Which one would you trust to find a *different* GameObject in the scene?
>
> _Continue reading once you've made a guess._

Press Play. In the Console you'll see (in order):

```
OnAwake on Cube
OnEnabled on Cube
OnStart on Cube
```

Stop the game and you'll see `OnDisabled` and `OnDestroy` fire as the scene tears down.

The order matters:

- `OnAwake` runs once, the very first time the component is initialized. Use it to grab references to *sibling* components — the GameObject already exists at this point.
- `OnEnabled` runs every time the component (or its parent GameObject) flips from disabled to enabled. It will be called again if you toggle `Enabled` later.
- `OnStart` runs **once before the first `OnUpdate`**. By the time it fires, every component in the scene has had its `OnAwake` called, so it's safe to look up other GameObjects you don't own.
- `OnUpdate` runs every rendered frame.
- `OnFixedUpdate` (not shown above) runs at the physics tick rate — use it for physics work, not rendering work.
- `OnDestroy` runs once when the component or its GameObject is destroyed.

> **Common error.** Putting `Scene.GetAllComponents<X>()` in `OnAwake` can return an incomplete list because not every GameObject has woken yet. Move that lookup to `OnStart`.

### Checkpoint

Add a second cube with `SpinComponent` to the scene. You should see the `OnAwake/OnEnabled/OnStart` triplet log twice when you press Play, once per instance. If you only see it once, you accidentally put the component on the same GameObject twice — check your Scene Tree.

---

## Step 4: Decorating properties

`[Property]` is what tells the editor "expose this in the Inspector". A handful of related attributes shape the UI:

```csharp
[Property, Range( 0, 720 )]
public float SpinSpeed { get; set; } = 90f;

[Property, Group( "Visuals" )]
public Color TintWhileSpinning { get; set; } = Color.White;

[Property, Title( "Use World Space Axis" )]
public bool WorldSpace { get; set; } = true;

[Property, Hide]
public Vector3 _internalAxisCache;
```

- `[Range( min, max )]` — clamps and shows a slider. The third overload `Range( min, max, step, clamped, slider )` lets you set step size, or turn the slider off if you only want clamping.
- `[Group("...")]` — groups properties under a collapsible header in the Inspector.
- `[Title("...")]` — overrides the auto-generated label.
- `[Hide]` — keeps the property serialized but hides it from the UI.

> **If your `[Property]` doesn't show up**, three things to check: the property has a `public` getter, both `get` and `set` exist (auto-property is fine), and the type is one s&box knows how to serialize (primitives, `Vector3`, `Color`, `GameObject`, `Component` references, etc).

---

## Step 5: Components talking to components

Components are most useful when they cooperate. Let's make `SpinComponent` change the model's tint while it spins.

```csharp
using Sandbox;

public sealed class SpinComponent : Component
{
    [Property, Range( 0, 720 )] public float SpinSpeed { get; set; } = 90f;
    [Property] public Color TintWhileSpinning { get; set; } = new Color( 1, 0.6f, 0.2f );

    private ModelRenderer _renderer;
    private Color _originalTint;

    protected override void OnStart()
    {
        // Look up a sibling component on the same GameObject.
        _renderer = Components.Get<ModelRenderer>();

        if ( _renderer is not null )
            _originalTint = _renderer.Tint;
    }

    protected override void OnUpdate()
    {
        GameObject.WorldRotation *= Rotation.FromAxis( Vector3.Up, SpinSpeed * Time.Delta );

        if ( _renderer is not null )
            _renderer.Tint = SpinSpeed > 0.01f ? TintWhileSpinning : _originalTint;
    }
}
```

> **Pause and predict:** We moved the `Components.Get<ModelRenderer>()` lookup into `OnStart`. What would change if we put it in `OnAwake` instead? And what about putting it directly in `OnUpdate` — would that even work?
>
> _Continue reading once you've made a guess._

`Components.Get<T>()` walks the GameObject's component list and returns the first one of that type, or `null` if none exists. There are sibling helpers:

- `Components.Get<T>( includeDisabled: true )` — also returns disabled components.
- `Components.GetAll<T>()` — returns every match, not just the first.
- `Components.GetInChildren<T>()` and `GetInParent<T>()` — walk the GameObject hierarchy.

Press Play. The cube tints orange while spinning; if you set `SpinSpeed` to 0 in the Inspector, it returns to its base color.

### `[RequireComponent]` vs runtime lookup

If your component genuinely doesn't make sense without another component present, use `[RequireComponent]`:

```csharp
public sealed class SpinComponent : Component
{
    [RequireComponent] public ModelRenderer Renderer { get; set; }
    // ...
}
```

When the Editor adds your component to a GameObject, it'll automatically add a `ModelRenderer` if there isn't one already, and the `Renderer` property gets populated for you. Use `[RequireComponent]` when the dependency is hard. Use `Components.Get<T>()` when the dependency is optional or runtime-discovered.

> **If you see a `NullReferenceException` on `_renderer.Tint`**, it means the GameObject didn't have a `ModelRenderer` when `OnStart` ran. Either add one, switch to `[RequireComponent]`, or guard with `if ( _renderer is not null )` like the example does.

---

## Step 6: A second component, talking to the first

Drop this in a new file `SpinObserver.cs`:

```csharp
using Sandbox;

public sealed class SpinObserver : Component
{
    [Property] public SpinComponent Target { get; set; }
    [Property] public float WarnAbove { get; set; } = 360f;

    protected override void OnUpdate()
    {
        if ( Target is null ) return;

        if ( Target.SpinSpeed > WarnAbove )
            Log.Warning( $"{Target.GameObject.Name} is spinning too fast: {Target.SpinSpeed}" );
    }
}
```

Add `SpinObserver` to a different GameObject. Drag the cube from the Scene Tree into the `Target` slot in the Inspector. Now press Play and crank the cube's `SpinSpeed` past 360 — the console fills with warnings.

`[Property] public SpinComponent Target` produces an object-picker slot in the Inspector. The serialized reference survives saves and is restored next time you load the scene.

---

## Final code

```csharp
using Sandbox;

public sealed class SpinComponent : Component
{
    [Property, Range( 0, 720 )] public float SpinSpeed { get; set; } = 90f;
    [Property] public Color TintWhileSpinning { get; set; } = new Color( 1, 0.6f, 0.2f );

    private ModelRenderer _renderer;
    private Color _originalTint;

    protected override void OnStart()
    {
        _renderer = Components.Get<ModelRenderer>();
        if ( _renderer is not null )
            _originalTint = _renderer.Tint;
    }

    protected override void OnUpdate()
    {
        GameObject.WorldRotation *= Rotation.FromAxis( Vector3.Up, SpinSpeed * Time.Delta );

        if ( _renderer is not null )
            _renderer.Tint = SpinSpeed > 0.01f ? TintWhileSpinning : _originalTint;
    }
}
```

## Troubleshooting

:::danger Object not spinning
First check the basics: did you press Play? Is `SpinSpeed > 0`? Is the component enabled in the Inspector (the checkbox next to its name)? If all yes, the GameObject's `OnUpdate` isn't running at all — check that the parent GameObject's `Enabled` is also true. A disabled parent suppresses every child component.
:::

:::warning Component doesn't appear in Add Component
A compile error somewhere in your project will block the new type from showing up. Open `View -> Console` and scroll to the most recent build. The build output also writes to your Editor log — you don't need to restart.
:::

:::warning Inspector property reverts when I press Play
If you set a property at runtime (during Play) and then stop, the Editor returns it to whatever the saved scene specified. To make a value stick, set it while the game is **stopped**, then save the scene.
:::

## Try extending it

- Make `SpinComponent` work on any axis. Add `[Property] public Vector3 Axis { get; set; } = Vector3.Up;` and pass it to `Rotation.FromAxis`.
- Replace the tint logic with a `[RequireComponent]` reference and remove the null check. See whether the Editor still lets you remove the renderer afterwards.
- Add `[Property] public bool RandomizeOnStart { get; set; }`. In `OnStart`, when set, pick a `SpinSpeed` between 30 and 360 with `Game.Random.Float( 30, 360 )`.

## Try it yourself

Now add a **fourth lifecycle hook** to `SpinComponent`: `OnFixedUpdate`. The tutorial mentioned it runs at the physics tick rate, but you've never actually watched it fire alongside the others. Your task:

- Add `OnFixedUpdate` next to the existing hooks, with a `Log.Info` like the others.
- **Before you press Play**, predict how often you'll see it log compared to `OnUpdate`. Will it print *more* often, *less* often, or about the same?
- Then add a counter that increments in `OnUpdate` and one in `OnFixedUpdate`, and log both every second so you can see the actual ratio.

You've used `[Property]` and `Log.Info` already. The new piece is `OnFixedUpdate` itself — it's a `protected override void` like the others.

Try it before reading on. Get stuck? Here's a hint:

<details>
<summary>Hint 1</summary>

Add a `TimeUntil _nextLog;` field, set it to `1f` in `OnStart`, and check it inside whichever hook you like — when `_nextLog` is true (it converts to bool), log the counters and reset to `1f`.

</details>

<details>
<summary>Hint 2 (the answer)</summary>

```csharp
private TimeUntil _nextLog;
private int _updateCount;
private int _fixedCount;

protected override void OnStart()
{
    Log.Info( $"OnStart on {GameObject.Name}" );
    _nextLog = 1f;
}

protected override void OnFixedUpdate()
{
    _fixedCount++;
}

protected override void OnUpdate()
{
    _updateCount++;
    GameObject.WorldRotation *= Rotation.FromAxis( Vector3.Up, SpinSpeed * Time.Delta );

    if ( _nextLog )
    {
        Log.Info( $"In the last second: OnUpdate fired {_updateCount}x, OnFixedUpdate fired {_fixedCount}x" );
        _updateCount = 0;
        _fixedCount = 0;
        _nextLog = 1f;
    }
}
```

You'll see `OnFixedUpdate` fire at roughly the engine's fixed tick rate (commonly ~50/sec) regardless of your monitor's refresh rate, while `OnUpdate` fires at your render frame rate. That's exactly why physics math should live in `OnFixedUpdate` — it's framerate-independent without needing `Time.Delta` adjustments.

</details>

## Next steps

- Read about the broader [Component Lifecycle](../scene/components/component-lifecycle.md) including `OnFixedUpdate`, `OnPreRender`, and `OnValidate`.
- Continue to [Build a Third-Person Controller](build-a-third-person-controller.md) for a non-trivial component combo.
- For multiplayer behavior on a component, see [Networking Basics](networking-basics.md).
