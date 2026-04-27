---
title: "Your First Game"
icon: "🎮"
sources:
  - engine/Sandbox.Engine/Scene/Components/Game/PlayerController/PlayerController.cs
  - engine/Sandbox.Engine/Scene/Components/Markers/ITriggerListener.cs
  - engine/Sandbox.Engine/Scene/Components/UI/PanelComponent.cs
updated: 2026-04-25
created: 2026-04-27
---

# Your First Game

A small, complete coin-collector built end to end. By the time you're done you'll have a player you can run around with, pickups that disappear when you touch them, a score, and a HUD. The point is to see how the engine's pieces — Components, GameObjects, Prefabs, Triggers, UI — fit together in one place.

It takes about half an hour the first time through, depending on how many side quests you take.

## What you'll build

A player character running on a flat arena, scattered spinning coins, a `GameManager` tracking score, and a Razor panel showing the score on screen. Every interaction in the engine you'll see again later — this is the canonical "everything talks to everything" example.

## Prerequisites

- s&box installed and the Editor opens.
- Some C# familiarity (you don't need to be a wizard, but you should recognise `class`, `protected override`, and properties).

---

## Step 1: Project setup

Open the Editor. From the welcome screen pick **Create New Project** and choose the **Game** template. Name it `CoinCollector`.

The project comes up with an empty scene called `untitled.scene`. Save it now: **File -> Save Scene As...** -> `main.scene`. Saving early means hot-reloads have somewhere to write to.

> If the welcome screen doesn't show templates, your install might be missing the `facepunch.base` content addon. Check the launcher's content tab — without `facepunch.base` you also won't have `models/dev/` paths used later in this tutorial.

### Anatomy of a `.sbproj`

Open your project's folder (right-click the project in the Editor and pick **Open in Explorer**). You'll see a `CoinCollector.sbproj` file. It's plain JSON — open it in a text editor:

```json
{
  "Title": "CoinCollector",
  "Type": "game",
  "Schema": 1,
  "ResourcePaths": [ "/" ],
  "CodePath": "Code/",
  "PackageReferences": [ "facepunch.base" ]
}
```

You don't need to edit this manually for the tutorial — the Editor manages it — but knowing it exists helps when something goes missing later.

### Checkpoint

You should see an empty 3D viewport with a grid, a Scene Tree on the left, and an Inspector on the right. The window title should read something like `main.scene - CoinCollector`. If anything else, save the scene and try again.

---

## Step 2: The arena

We need a floor to stand on and a sky to look at.

1. Right-click the **Scene Tree** -> **Create Object -> 3D Object -> Cube**. The Cube comes preloaded with a `ModelRenderer` and a `BoxCollider`.
2. Rename it `Floor` (F2 or click again on its name).
3. With `Floor` selected, look at the Inspector. Set `WorldScale` to `(2000, 2000, 10)`. The model is `models/dev/box.vmdl` by default — those `dev/` models are the engine's debug primitives, perfect for blockouts.
4. The default `WorldPosition` is `(0, 0, 0)`. Leave it.

> **If the cube doesn't appear in the viewport**, your camera is probably below it. Press `F` with the cube selected to focus the editor camera on it.

### Checkpoint

The viewport now shows a large grey checker-textured slab. Save (`Ctrl+S`).

---

## Step 3: Light

Without a light source the scene renders black at runtime.

1. Right-click the Scene Tree -> **Create Empty GameObject** -> rename `Sun`.
2. With `Sun` selected, **Add Component** -> `DirectionalLight`. This is a global "sun-style" light: direction matters, position doesn't.
3. Set `Sun`'s `WorldRotation` to `(45, 45, 0)` so the light comes from above and to the side.
4. Create another empty GameObject named `Environment`. Add an `EnvmapProbe` to it. The envmap probe captures ambient light so shadowed surfaces aren't pitch black, and is what gives metallic surfaces their reflections.

> **If the floor still looks flat-lit**, the `EnvmapProbe` probably hasn't built. With it selected, click **Bake** in the Inspector — or just press Play once and stop, which forces a rebuild.

### Checkpoint

Floor now has visible shading — one side should be brighter than the other. If everything is uniformly grey, the directional light isn't pointing at it (re-check the rotation).

---

## Step 4: The player

Rather than write a movement controller from scratch, we'll drop in `PlayerController` — the same component used in most s&box example games.

1. **Create Empty GameObject** named `Player`.
2. Set its `WorldPosition` to `(0, 0, 50)` so it spawns above the floor instead of inside it.
3. **Add Component** -> `PlayerController`. This single component handles WASD movement, jumping, gravity, ducking, and an optional camera.
4. The player needs a camera. Right-click `Player` -> **Create Empty Child** -> rename `Camera`.
5. Select `Camera` and **Add Component** -> `CameraComponent`.
6. Back on `Player`, look at the `PlayerController` properties in the Inspector. Under the **Camera** group, leave **Use Camera Controls** ticked. The default `CameraOffset` of `(256, 0, 12)` is third-person; if you want first-person, untick **Third Person**.

`PlayerController` finds the active `CameraComponent` automatically — you don't have to drag a reference into a slot. It looks at `Scene.Camera`.

> **If the camera doesn't follow the player at runtime**, you have either no `CameraComponent` in the scene, or `Use Camera Controls` is unchecked. Both are silent failures — there's no error in the console.

### Checkpoint

Press **Play**. WASD moves you, mouse looks. Press Space to jump. The camera trails behind. If you fall through the floor, your `Floor` lost its `BoxCollider` — re-add it.

Stop the game (Esc, or click the stop button).

---

## Step 5: A spinning, collectible coin

Now an item the player can pick up.

1. Create an empty GameObject named `Coin`.
2. Add a `ModelRenderer`. Set **Model** to `models/dev/cylinder.vmdl`. (The `models/dev/` family ships with `facepunch.base`; using one means no asset import.)
3. Set `WorldScale` to `(0.5, 0.5, 0.1)` so it's a flat disc.
4. Add a `BoxCollider`. **Tick the `Is Trigger` checkbox.** A trigger collider doesn't physically block — it just fires events when other colliders enter or leave it. That's what makes it a pickup instead of a wall.

### Make it spin

> **Pause and predict:** The coin needs to spin. If we wrote `GameObject.LocalRotation = Rotation.FromYaw( Speed * Time.Delta )` (note the `=` instead of `*=`), what would happen at runtime? And why is the multiplication by `Time.Delta` there at all?
>
> _Continue reading once you've made a guess._

In the Assets Browser, **Create -> C# Component**, name `CoinSpin.cs`:

```csharp
using Sandbox;

public sealed class CoinSpin : Component
{
    [Property] public float Speed { get; set; } = 90f;

    protected override void OnUpdate()
    {
        // LocalRotation here, not WorldRotation - so spinning works regardless of the parent's transform
        GameObject.LocalRotation *= Rotation.FromYaw( Speed * Time.Delta );
    }
}
```

Save, wait for the green check, then add `CoinSpin` to your `Coin`. Press Play — it spins.

> **If the coin doesn't spin**, the most likely cause is that you wrote `WorldRotation` and parented the coin under something that re-set rotation each frame (none of our objects do that yet, but it's a recurring gotcha as scenes grow). `LocalRotation` is the safer default for self-driven motion.

### Make it a Prefab

We want lots of coins, so we extract this one into a reusable Prefab.

1. Drag the `Coin` GameObject from the Scene Tree down into the Assets Browser. The Editor saves a `Coin.prefab` file.
2. Delete the original `Coin` from the scene (it's now a Prefab; the in-scene copy is no longer needed).
3. Drag a few copies of `Coin.prefab` from the Assets Browser into the viewport, spreading them out around the floor.

Each in-scene Prefab instance is a live link back to the source — edit the prefab, every instance updates.

### Checkpoint

Press Play. You can run between coins; they spin. They don't disappear yet — that's next.

---

## Step 6: Score and pickup

Two new components: a `GameManager` that owns the score, and a `Collectible` that detects the player.

`GameManager.cs`:

```csharp
using Sandbox;

public sealed class GameManager : Component
{
    public static int Score { get; private set; }

    protected override void OnStart()
    {
        Score = 0;
    }

    public static void AddScore( int amount = 10 )
    {
        Score += amount;
        Log.Info( $"Score is now: {Score}" );
    }
}
```

Create an empty GameObject in the scene called `GameManager` and attach the `GameManager` component to it.

> **Why a static `Score`?** Quick and dirty. It works because there's exactly one `GameManager` and one local player. In multiplayer you'd put `Score` on the player's own GameObject and use `[Sync]` instead — see the [Networking Basics](networking-basics.md) tutorial.

> **Pause and predict:** The `Collectible` we're about to write needs to detect when the player walks into a coin. Given that we ticked `Is Trigger` on the coin's `BoxCollider` and the player has the `PlayerController` component, what method do you think will fire — and on which GameObject's component? What will it receive as an argument?
>
> _Continue reading once you've made a guess._

`Collectible.cs`:

```csharp
using Sandbox;

public sealed class Collectible : Component, Component.ITriggerListener
{
    [Property] public int Value { get; set; } = 10;

    public void OnTriggerEnter( Collider other )
    {
        // Only count the player. Ignore other coins, debris, etc.
        var player = other.GameObject.Components.Get<PlayerController>();
        if ( player is null ) return;

        GameManager.AddScore( Value );
        GameObject.Destroy();
    }

    public void OnTriggerExit( Collider other ) { }
}
```

Open `Coin.prefab` (double-click). Add the `Collectible` component to its root. Save the prefab — every instance in the scene picks up the change immediately because of the live-link.

> **If `OnTriggerEnter` never fires**, the usual suspect is a missing `BoxCollider` on the player. `PlayerController` does *not* implicitly have a collider — it has its own internal trace shape, but the trigger system still wants the *other* side to be a collider too. The `PlayerController` template adds one for you on creation; if you removed it, add a `BoxCollider` back.

### Checkpoint

Press Play. Walk into a coin. The console logs `Score is now: 10` and the coin disappears. Pick up another — `20`. Pick them all up — only the GameManager remains.

---

## Step 7: HUD

Console output is satisfying for the developer but useless to a player. Time for a Razor panel.

1. Assets Browser -> **Create -> Razor Panel Component** -> name `ScoreUI.razor`.
2. Replace its contents with:

```html
@using Sandbox;
@using Sandbox.UI;
@inherits PanelComponent

<root>
    <div class="score-container">
        <label>Score: @GameManager.Score</label>
    </div>
</root>

@code
{
    // Razor panels redraw only when this hash changes. Including Score here means
    // the label updates whenever Score does.
    protected override int BuildHash() => System.HashCode.Combine( GameManager.Score );
}

<style>
    .score-container {
        position: absolute;
        top: 50px;
        left: 50px;
        font-size: 48px;
        color: white;
        font-family: Poppins;
        font-weight: bold;
        text-shadow: 2px 2px 4px rgba( 0, 0, 0, 0.5 );
    }
</style>
```

3. Select the `Camera` GameObject inside `Player`. **Add Component** -> `ScreenPanel`. This is the canvas that hosts UI on the screen.
4. **Add Component** -> `ScoreUI`. The panel parents itself under the `ScreenPanel`.

> **If the UI doesn't appear**, double-check both `ScreenPanel` and `ScoreUI` are on the same GameObject (or `ScoreUI` is on a child of a GameObject with `ScreenPanel`). Without the parent `ScreenPanel`, `PanelComponent`-derived components have no root to render into.

> **If the score doesn't update**, the `BuildHash` method has to include every value the panel reads. Forget it, and you'll get the initial value frozen on screen forever.

### Checkpoint

Press Play. "Score: 0" appears in the top-left. Walk into a coin, the number jumps to 10. The text font is Poppins, which ships with the engine; if you see a generic-looking font instead, you may have a typo in the `font-family` line — it falls back silently.

---

## Step 8: Play it

Run around. Collect coins. Watch the score climb.

You just used:

- The **Scene Tree** for hierarchy and the Inspector for per-object properties.
- Built-in components (`PlayerController`, `CameraComponent`, `ModelRenderer`, `BoxCollider`, `DirectionalLight`, `EnvmapProbe`) plus a `ScreenPanel` for UI.
- Custom components (`CoinSpin`, `GameManager`, `Collectible`).
- A **Prefab** for asset reuse.
- A **Trigger** for non-blocking detection (`ITriggerListener.OnTriggerEnter`).
- A **Razor Panel** for the HUD with `BuildHash`-driven redraw.

## Performance note

`OnUpdate` runs every rendered frame, so 144 Hz monitors run it more than 60 Hz monitors. Always multiply by `Time.Delta` for anything frame-rate-sensitive (rotation, movement). The `CoinSpin` does this; if you forget it, coins spin twice as fast on faster monitors.

## Troubleshooting

:::danger Player falls through the floor
The `Floor`'s `BoxCollider` got removed or the `IsTrigger` was checked. The PlayerController needs a solid (non-trigger) collider underneath.
:::

:::warning Coins don't disappear
Check three things in order: (1) the coin's `BoxCollider` has `IsTrigger` ticked; (2) the `Collectible` component is on the coin prefab's root; (3) the Player has a `BoxCollider` somewhere. Walk into a coin and watch the Console — if `Score is now:` doesn't print, the trigger never fired.
:::

:::warning UI shows but score doesn't update
`BuildHash()` is missing the `Score` field, or you forgot to override it entirely. The default returns 0, which means "never redraw".
:::

:::tip "I see the score stuck at 0 even though the console logs go up"
Same as above. Triple-check `System.HashCode.Combine( GameManager.Score )`.
:::

## Try extending it

- **Sound on pickup.** In `Collectible.OnTriggerEnter`, before `Destroy()`, call `Sound.Play( "ui.button.press" )` (replace with any sound you have, or import your own `.sound` asset).
- **Limited time.** Add a `[Property] public float RoundDuration = 30f;` to `GameManager`, count down in `OnUpdate`, and freeze input when time runs out.
- **Bigger coins worth more.** The `Collectible` already has a `Value` property — make a "gold coin" prefab variant by duplicating `Coin.prefab`, scaling it up, and setting `Value = 100`.
- **Per-coin spin speed.** `CoinSpin.Speed` is already a `[Property]`, so each coin can be set differently in the Inspector. Try randomizing it in `OnStart`: `Speed = Game.Random.Float( 60, 180 );`.

## Try it yourself

Now extend the game with a **second collectable type** — a "gem" worth 50 points instead of 10. This builds directly on what you just did:

- Duplicate `Coin.prefab` and rename it `Gem.prefab`. Change its `ModelRenderer.Model` to something different (e.g. `models/dev/sphere.vmdl`) so you can tell them apart.
- The `Collectible` component already has a `Value` property — set it to `50` on the gem prefab in the Inspector.
- Drop a few gem instances next to the coins in the scene.

That's actually the whole task — no new code required. But predict before you try: when you pick up a gem, will the score jump by 50, or will it still go up by 10? Why?

Try it before reading on. Get stuck on the prediction? Here's a hint:

<details>
<summary>Hint 1</summary>

Look at `Collectible.OnTriggerEnter`. It calls `GameManager.AddScore( Value )`. The `Value` it passes is the *instance's* `Value` property — the one you set per-prefab in the Inspector. Each prefab variant is a separate file with its own serialized property values.

</details>

<details>
<summary>Hint 2 (the answer)</summary>

The score jumps by **50** for gems and **10** for coins. The `[Property] public int Value { get; set; } = 10;` on `Collectible` declares an instance-level field, so each prefab (and each in-scene instance) carries its own value. The `GameManager.AddScore( Value )` call inside `OnTriggerEnter` passes whatever `Value` the specific instance has.

If you wanted to be surer about which one was picked up, you could log it:

```csharp
public void OnTriggerEnter( Collider other )
{
    var player = other.GameObject.Components.Get<PlayerController>();
    if ( player is null ) return;

    Log.Info( $"Picked up {GameObject.Name} worth {Value}" );
    GameManager.AddScore( Value );
    GameObject.Destroy();
}
```

</details>

## Where to go next

- [Build Your First Component](your-first-component.md) — the line-by-line tour of what `Component` actually does.
- [Build a Third-Person Controller](build-a-third-person-controller.md) — extending `PlayerController` instead of using it as-is.
- [Build a UI](build-a-ui.md) — Razor panels in proper depth, including data-binding and styling.
