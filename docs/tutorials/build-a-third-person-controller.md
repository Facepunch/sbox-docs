---
title: "Build a Third-Person Controller"
icon: "📹"
sources:
  - engine/Sandbox.Engine/Scene/Components/Game/PlayerController/PlayerController.cs
  - engine/Sandbox.Engine/Scene/Components/Game/PlayerController/PlayerController.Camera.cs
  - engine/Sandbox.Engine/Scene/Components/Game/PlayerController/PlayerController.Input.cs
  - engine/Sandbox.Engine/Scene/Components/Camera/CameraComponent.cs
updated: 2026-04-25
created: 2026-04-27
---

# Build a Third-Person Controller

`PlayerController` is the engine's batteries-included character: WASD, jumping, crouching, third-person camera with auto-zoom, animator hookups. This walkthrough sets one up in the Editor, then writes a small companion component that adds health and a double-jump on top — because you'll almost always extend the controller rather than rewrite it.

## What you'll build

A `Player` GameObject with:

- `ModelRenderer` showing a Citizen model
- `PlayerController` for movement and camera
- A child `Camera` with a `CameraComponent`
- A custom `PlayerHealth` component you'll write

By the end, WASD moves, mouse looks, Space jumps (twice), and the player has health that ticks down on a timer.

## Prerequisites

- Comfortable with [Your First Component](your-first-component.md).
- A scene with a floor and some lighting (the [Your First Game](your-first-game.md) arena works perfectly).

---

## Step 1: Create the Player

In the Scene Tree, right-click and pick **Create Empty GameObject**. Name it `Player`. Set its `WorldPosition` to `(0, 0, 50)` so it spawns above the floor.

Don't add anything else yet. We'll layer components in the order they make sense.

> **If you can't find "Create Empty GameObject"**, you might be right-clicking on an existing GameObject — that produces a different menu. Right-click in the empty space below the tree.

### Checkpoint

Press Play. Nothing happens — the player has no visual or physics yet, but you can hear a faint "no error" silence. Stop the game.

---

## Step 2: Give it a body

`PlayerController` doesn't render anything; it just moves a transform. We need a model.

1. Select `Player`. **Add Component** -> `ModelRenderer`.
2. Set its **Model** to `models/citizen/citizen.vmdl`. The Citizen is the default humanoid that ships with the engine.
3. The model spawns rooted at the GameObject origin, head pointing up. By default the GameObject's pivot sits at the model's feet — convenient for walking.

> **If the Citizen model isn't in your asset list**, your project's `.sbproj` might not be referencing `facepunch.base`. Open the project in Explorer, edit the `.sbproj`, and ensure `"facepunch.base"` is in `PackageReferences`. Restart the Editor.

> **If you see a checkerboard "missing" model**, the path is wrong — the typical mistake is `models/Citizen/...` (capital C) which won't resolve on case-sensitive paths.

### Checkpoint

A blue-jumpsuited figure appears at the origin. Save the scene.

---

## Step 3: Drop in the PlayerController

1. With `Player` still selected, **Add Component** -> `PlayerController`.

The Inspector reveals a long list of grouped properties: **Movement**, **Input**, **Camera**, **Animation**, **Footsteps**. The defaults are tuned for a humanoid — leave them alone for now.

A few you'll care about later:

- `Body` — a reference to the `SkinnedModelRenderer` (or `ModelRenderer`) that should be hidden in first-person. Set it to your `ModelRenderer` if you want toggleable POV.
- `EyeDistanceFromTop` — how far below the top of the controller the eyes sit (default 8 units).
- `ToggleCameraModeButton` — the input action that flips first/third person at runtime. Default is `"view"`.

> **If the PlayerController complains about missing input bindings** in the console, the project doesn't have its `Input.Actions` populated. Open **Project Settings -> Input** and confirm `view`, `forward`, `back`, `left`, `right`, `jump` exist. New projects from the **Game** template have these by default.

---

## Step 4: Camera

The PlayerController can drive any active `CameraComponent` in the scene — it doesn't need a direct reference because it asks `Scene.Camera` for the active camera each frame.

1. Right-click `Player` in the Scene Tree -> **Create Empty Child** -> name it `Camera`.
2. Select `Camera`. **Add Component** -> `CameraComponent`.

That's it. When you press Play, the controller's `UpdateCameraPosition` (in `PlayerController.Camera.cs`) takes over: it sets the camera's `WorldPosition` and `WorldRotation` every frame and adds `viewer` to its `RenderExcludeTags` so the player's own body doesn't clip the view.

### Configure the camera offset

On the `PlayerController` component, expand the **Camera** group:

- **Use Camera Controls** — checked. Untick this to take manual control.
- **Third Person** — checked. Default is third person; the `view` input toggles it.
- **Camera Offset** — `(256, 0, 12)`. That means 256 units behind, 0 sideways, 12 units up from the eye position. Increase the X for a more cinematic distance, increase Z to look down on the character.
- **Use Fov From Preferences** — checked, uses the player's preferred FOV.
- **Hide Body In First Person** — checked, hides the assigned `Body` renderer when first-person is active.

> **If the camera doesn't follow the player**, either there's no `CameraComponent` enabled in the scene, or `Use Camera Controls` is off. There's no error logged for this — it just silently doesn't update.

> **If the camera clips through walls**, that's actually working as designed: the controller traces from the eye position outward and shortens the distance when something gets in the way. If it feels too jumpy, increase the smoothing in your own derived camera logic — `PlayerController.Camera.cs` lerps `_cameraDistance` toward the trace hit at varying speeds depending on whether the camera was started inside a wall.

### Checkpoint

Press Play. WASD moves the Citizen, mouse looks, Space jumps. The camera trails 256 units behind. Walk up to a wall and turn — the camera should pull in close to keep the player visible.

If you press the `view` input (default keybind: bind whatever you've set, often `V`), it switches to first-person.

---

## Step 5: Add health (and a place to put it)

> **Pause and predict:** Why do you think `PlayerController` doesn't ship with a built-in `Health` field? What would go wrong if a generic engine controller decided how health worked for *every* game built on it?
>
> _Continue reading once you've made a guess._

The PlayerController is intentionally minimal about gameplay state — it doesn't know what health is, what hunger is, what your inventory looks like. That's your job. The convention is to write a *companion component* that lives next to the controller on the same GameObject and reads from it.

Create `PlayerHealth.cs` in your Code folder:

```csharp
using Sandbox;

public sealed class PlayerHealth : Component
{
    [Property, Range( 0, 200 )]
    public float MaxHealth { get; set; } = 100f;

    [Property]
    public float Current { get; private set; } = 100f;

    [Property]
    public bool TickDamageOverTime { get; set; } = false;

    [Property, ShowIf( nameof( TickDamageOverTime ), true )]
    public float DamagePerSecond { get; set; } = 5f;

    private TimeUntil _nextTick;

    protected override void OnStart()
    {
        Current = MaxHealth;
        _nextTick = 1f;
    }

    protected override void OnUpdate()
    {
        if ( !TickDamageOverTime ) return;
        if ( Current <= 0f ) return;

        if ( _nextTick )
        {
            _nextTick = 1f;
            TakeDamage( DamagePerSecond );
        }
    }

    public void TakeDamage( float amount )
    {
        Current = MathF.Max( 0f, Current - amount );

        if ( Current <= 0f )
            OnDeath();
    }

    public void Heal( float amount )
    {
        Current = MathF.Min( MaxHealth, Current + amount );
    }

    void OnDeath()
    {
        Log.Info( $"{GameObject.Name} died" );
        // Disable the controller so we stop moving on the way down.
        var controller = Components.Get<PlayerController>();
        if ( controller is not null )
            controller.Enabled = false;
    }
}
```

Add `PlayerHealth` to your `Player` GameObject. In the Inspector, tick **Tick Damage Over Time** so we can watch it work.

> **If `[ShowIf]` doesn't compile**, you imported the wrong `Sandbox` namespace — make sure it's `using Sandbox;` at the top, not a custom one.

### Checkpoint

Press Play. Watch the `Current` field in the Inspector tick down by 5 every second. After 20 seconds it hits zero, the death log fires, and the character stops responding to input because we disabled the controller.

---

## Step 6: Double-jump

For the small win, let's add a double-jump that triggers on the second press of `jump`. We'll do this with a second small component so the controller stays vanilla.

> **Pause and predict:** We're going to put `PlayerDoubleJump` on the *same* GameObject as `PlayerController`, not a child. Both override `OnUpdate`. When you press the jump key, will the standard PlayerController jump fire *and* our bonus jump fire on the same press, or only one? What stops them from doubling up on the very first jump?
>
> _Continue reading once you've made a guess._

`PlayerDoubleJump.cs`:

```csharp
using Sandbox;

public sealed class PlayerDoubleJump : Component
{
    [Property] public float JumpStrength { get; set; } = 350f;

    PlayerController _controller;
    int _jumpsRemaining;

    protected override void OnStart()
    {
        _controller = Components.Get<PlayerController>();
    }

    protected override void OnUpdate()
    {
        if ( _controller is null ) return;

        // Reset when we land
        if ( !_controller.IsAirborne )
        {
            _jumpsRemaining = 1; // one bonus jump while airborne
            return;
        }

        if ( _jumpsRemaining <= 0 ) return;

        if ( Input.Pressed( "jump" ) )
        {
            // PlayerController.Jump applies an upward kick using its standard logic:
            // it cancels opposing velocity (so you don't get a free pop while falling)
            // and clamps the resulting velocity to the magnitude of what you pass in.
            _controller.Jump( Vector3.Up * JumpStrength );
            _jumpsRemaining--;
        }
    }
}
```

Add `PlayerDoubleJump` to the same `Player` GameObject. Press Play.

> **If the second jump doesn't fire**, check that `_controller.IsAirborne` is what you expect. The controller exposes ground-check state that we read here — if you're standing on a slope past `GroundAngle` (45° default), you might already be considered airborne and your "first" jump is being consumed.

> **If the second jump feels weak**, that's `Jump(Vector3)` clamping the resulting velocity to the magnitude of the input. Pass a larger vector — e.g., `Vector3.Up * 500f` — for a higher hop. Don't mutate `_controller.Velocity` directly; the setter is private.

### Checkpoint

WASD around the floor. First jump uses the standard `PlayerController` jump; in mid-air, press jump again — you get a second one. Land, jump again, and you have your bonus back.

---

## Step 7: Programmatic spawn (optional)

If you'd rather build the player at runtime instead of placing it in the scene, the same setup looks like this:

```csharp
using Sandbox;

public sealed class PlayerSpawner : Component
{
    [Property] public Vector3 SpawnPosition { get; set; } = new Vector3( 0, 0, 50 );

    protected override void OnStart()
    {
        var go = new GameObject( true, "Player" );
        go.WorldPosition = SpawnPosition;

        var renderer = go.Components.Create<ModelRenderer>();
        renderer.Model = Model.Load( "models/citizen/citizen.vmdl" );

        var controller = go.Components.Create<PlayerController>();
        controller.UseCameraControls = true;
        controller.ThirdPerson = true;
        controller.CameraOffset = new Vector3( 256, 0, 12 );

        // Camera child
        var cam = new GameObject( true, "Camera" );
        cam.SetParent( go );
        cam.Components.Create<CameraComponent>();

        go.Components.Create<PlayerHealth>();
        go.Components.Create<PlayerDoubleJump>();
    }
}
```

Drop a `PlayerSpawner` component on any empty GameObject in the scene; remove the manually-placed `Player`. On Play, the spawner builds the same configuration in code.

This pattern is the one most multiplayer games use — see [Networking Basics](networking-basics.md), where each connecting client is given a fresh player programmatically.

---

## Troubleshooting

:::danger Player falls through the floor
The floor is a trigger, missing a collider, or below z=0 with the player above it. Confirm `Floor`'s `BoxCollider` exists and `IsTrigger` is **un**checked.
:::

:::warning Camera doesn't follow the player
You forgot to add a `CameraComponent` somewhere in the scene, or the one you added is on a disabled GameObject. The `PlayerController` queries `Scene.Camera`, which returns the first enabled camera.
:::

:::warning Citizen model looks wrong (T-pose, no animation)
`PlayerController` does *not* automatically drive animations. To get walk/run animations, you need a `CitizenAnimationHelper` (or your own animator). The controller's `Animation` group provides the input data — animation, however, lives in a separate component.
:::

:::tip Want to swap to first-person at startup
Untick `Third Person` in the Inspector, or set `controller.ThirdPerson = false` in `OnStart`. The toggle is also exposed via input action — see `ToggleCameraModeButton`.
:::

## Try extending it

- **Aim mode.** Press right mouse to pull the camera offset to `(80, 30, 0)` (closer, over-the-shoulder), release to restore. Tween it with `Vector3.Lerp` for smoothness.
- **Health regen.** In `PlayerHealth.OnUpdate`, when no damage has been taken for 5 seconds, regenerate 10/sec. Use a `TimeSince` field that resets in `TakeDamage`.
- **Damage volumes.** Make a `BoxCollider` with `IsTrigger`, attach an `ITriggerListener` component that calls `health.TakeDamage(20)` on enter — instant lava pits.
- **Per-instance jump tuning.** `PlayerDoubleJump.JumpStrength` is already a `[Property]`. Make `MaxJumps` configurable too, so you can have a triple-jump variant.

## Try it yourself

Now add a **camera shake when health drops**. The brief: when `PlayerHealth.TakeDamage` is called, the camera should jolt for a fraction of a second. This builds on what you've already used:

- `PlayerHealth.TakeDamage` is your hook — modify it (or add a small flag the camera reads).
- The camera lives on the child `Camera` GameObject with a `CameraComponent`. You can write a new component on that GameObject, perturb its `LocalPosition`, and let the controller's per-frame `UpdateCameraPosition` overwrite it back to neutral the next frame — or you can apply the shake offset in your own `OnUpdate` after the controller runs (`OnUpdate` order is generally creation order on the GameObject, but you can also fight it with a `LateUpdate`-style pattern).
- You've already used `TimeSince` (in the health regen extension idea) and `Vector3` math.

The simplest version: add a `ShakeAmount` field to a new `CameraShake` component on the camera, decay it over time, and offset `LocalPosition` by `Vector3.Random * ShakeAmount`.

Try it before reading on. Get stuck? Here's a hint:

<details>
<summary>Hint 1</summary>

In `PlayerHealth.TakeDamage`, find the camera shake component and bump its `ShakeAmount`:

```csharp
var shake = Components.GetInChildren<CameraShake>();
if ( shake is not null ) shake.ShakeAmount = 5f;
```

Then in `CameraShake.OnUpdate`, decay `ShakeAmount` toward 0 each frame and apply `Vector3.Random * ShakeAmount` to `LocalPosition`.

</details>

<details>
<summary>Hint 2 (the answer)</summary>

```csharp
using Sandbox;

public sealed class CameraShake : Component
{
    [Property] public float Decay { get; set; } = 20f;
    public float ShakeAmount { get; set; }

    protected override void OnUpdate()
    {
        // Decay toward zero
        ShakeAmount = MathF.Max( 0f, ShakeAmount - Decay * Time.Delta );

        // Random offset, scaled by current shake. LocalPosition because the
        // PlayerController writes WorldPosition; ours stays relative.
        GameObject.LocalPosition = Vector3.Random * ShakeAmount;
    }
}
```

And modify `PlayerHealth.TakeDamage`:

```csharp
public void TakeDamage( float amount )
{
    Current = MathF.Max( 0f, Current - amount );

    var shake = Components.GetInChildren<CameraShake>();
    if ( shake is not null ) shake.ShakeAmount = MathF.Min( 8f, shake.ShakeAmount + amount * 0.2f );

    if ( Current <= 0f )
        OnDeath();
}
```

Two subtle points worth noticing:
- We use `LocalPosition` because the `PlayerController` writes the camera's `WorldPosition` each frame from its own logic. Setting `LocalPosition` on the camera is a relative offset *within* whatever the parent transform is doing — so the shake survives.
- `Components.GetInChildren<CameraShake>()` walks down from the player to find the camera child, mirroring how `PlayerController` itself looks up `Body`.

</details>

## Next steps

- The full set of `PlayerController` properties is documented at the [Player Controller Reference](../scene/components/reference/player-controller.md).
- For interaction (picking up, pressing buttons), see [PlayerController.Pressing](../scene/components/reference/player-controller.md#interacting-with-objects-pressing).
- Multiplayer-ready player spawning is covered in [Networking Basics](networking-basics.md).
