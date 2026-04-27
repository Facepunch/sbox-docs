---
title: "Build a User Interface"
icon: "🖥️"
sources:
  - engine/Sandbox.Engine/Scene/Components/UI/PanelComponent.cs
  - engine/Sandbox.Engine/Scene/Components/UI/ScreenPanel.cs
updated: 2026-04-25
created: 2026-04-27
---

# Build a User Interface

> **New to UI?** This page is the **tutorial** for building a Razor HUD end-to-end. The full UI documentation is at **[UI](../systems/ui/index.md)**.

A HUD with a health bar, a damage button, and a low-health warning that flashes — built with Razor panels. Razor in s&box is the same templating dialect as ASP.NET Core, but rendered to the engine's UI layer instead of HTML. You write markup with embedded C# in `.razor` files; the engine compiles them into `PanelComponent`s.

## What you'll build

`HealthHud.razor` — a `PanelComponent` that:

- Displays current and max HP as text.
- Renders a health bar that fills proportionally.
- Has a button to take damage.
- Conditionally shows a "DANGER" warning when HP is below 25.
- Re-renders only when its data changes (no per-frame DOM thrashing).

## Prerequisites

- [Your First Component](your-first-component.md) — the Razor panel is a Component, just one with markup.
- Some HTML/CSS familiarity helps but isn't required. The CSS subset s&box supports is similar to flexbox-era browser CSS.

---

## Step 1: Create the file

In the Assets Browser, right-click and pick **Create -> Razor Panel Component**. Name it `HealthHud.razor`.

The file extension is `.razor`. The Editor compiles `.razor` files into `PanelComponent`-derived classes the same way it compiles `.cs` files into `Component` types.

> **If "Razor Panel Component" isn't in the create menu**, your project's `.sbproj` doesn't reference an addon that registers Razor templates. The default game project template enables this. Open the `.sbproj` and confirm you have Razor support; otherwise create from a template that does (the **Game** template is a safe choice).

---

## Step 2: Minimal markup

Open `HealthHud.razor` and replace its content:

```html
@using Sandbox;
@using Sandbox.UI;
@inherits PanelComponent

<root>
    <label>Hello UI</label>
</root>

@code {
}
```

A walk-through of what's there:

- `@using ...` — same as a C# `using` directive. `Sandbox` and `Sandbox.UI` cover the types you'll touch most.
- `@inherits PanelComponent` — the generated class derives from `PanelComponent`, which inherits from `Component`. That's what makes the file droppable on a GameObject.
- `<root>` — the implicit top-level element of the panel. The engine creates this even if you don't write it, but writing it makes the styling intent clear.
- `@code { }` — a C# class body. Anything you put inside lives on the generated class as fields, properties, and methods.

### Hook it up in the scene

1. Right-click the Scene Tree -> **Create Empty GameObject** -> name `HUD`.
2. Add a `ScreenPanel` to it. The `ScreenPanel` is a `Component` that registers itself as a screen-space root for any panels in its GameObject's children — without a `ScreenPanel` (or `WorldPanel`) ancestor, your `PanelComponent` has nowhere to render.
3. Add `HealthHud` to the same GameObject.

### Checkpoint

Press Play. You should see "Hello UI" in white text in the top-left corner. If you don't:

- The `ScreenPanel` is missing — add it.
- The `HealthHud` is on a different GameObject than the `ScreenPanel` — they need to share the GameObject (or the panel needs to be a child).
- A compile error blocks the panel — check the console.

> **If you see "Hello UI" but it's tiny**, that's the default font size with no styling. We'll fix it next.

---

## Step 3: Bind a value

We want the HUD to read the player's health. For now, we'll fake it — give the panel its own `Current` and `Max` properties so the rest of the game can write into them.

```html
@using Sandbox;
@using Sandbox.UI;
@inherits PanelComponent

<root>
    <div class="health-bar">
        <label class="hp-text">HP @Current / @Max</label>
    </div>
</root>

@code {
    [Property] public float Current { get; set; } = 100f;
    [Property] public float Max { get; set; } = 100f;

    protected override int BuildHash() => System.HashCode.Combine( Current, Max );
}
```

> **Pause and predict:** The `BuildHash` method returns a hash combining `Current` and `Max`. What happens if you forget to include `Max` in `BuildHash` and then change `Max` at runtime — do the labels show the new value, the old one, or something else? And what if `BuildHash` returns `0` (the default if you don't override it)?
>
> _Continue reading once you've made a guess._

`@Current` and `@Max` are interpolated values. Whenever the panel rebuilds, those expressions are evaluated and inserted as text.

But the panel only rebuilds when the value returned from `BuildHash()` changes — that's a deliberate optimization. If we didn't include both `Current` and `Max` in the hash, changes to `Max` would be invisible.

> **If `[Property]` doesn't appear in the Inspector**, check that `@inherits PanelComponent` is the first non-`@using` line. The base class is what gives you the `[Property]` machinery.

### Checkpoint

Save. The Editor recompiles. Look at the `HealthHud` component in the Inspector — you should see `Current` and `Max` fields. Press Play; the label reads "HP 100 / 100". Stop. Change `Current` to 75 in the Inspector. Press Play again; it now reads "HP 75 / 100".

---

## Step 4: A real health bar

Text is one thing; a fill bar is another. We'll set the inner bar's width as a percentage of the outer container.

```html
@using Sandbox;
@using Sandbox.UI;
@inherits PanelComponent

<root>
    <div class="health-frame">
        <div class="health-fill" style="width: @(FillPercent)%"></div>
        <label class="hp-text">HP @Current / @Max</label>
    </div>

    <button class="damage-btn" onclick="@OnDamageClicked">Take 10</button>
</root>

@code {
    [Property] public float Current { get; set; } = 100f;
    [Property] public float Max { get; set; } = 100f;

    public int FillPercent => Max > 0
        ? (int)(Current / Max * 100f).Clamp( 0, 100 )
        : 0;

    void OnDamageClicked()
    {
        Current = MathF.Max( 0f, Current - 10f );
    }

    protected override int BuildHash() => System.HashCode.Combine( Current, Max );
}
```

What changed:

- `style="width: @(FillPercent)%"` — inline style with an interpolated percentage. The `@()` form is needed when the expression is followed immediately by literal text (`%`); otherwise the parser would misread the boundary.
- `<button onclick="@OnDamageClicked">` — clicking calls the method. Razor wires up the event binding when the markup is built.
- `OnDamageClicked` — a normal C# method. It mutates `Current`, which changes the hash, which triggers a rebuild.

We still need styling to make any of this visible.

> **If clicking the button does nothing**, you might have written `onclick="OnDamageClicked"` (without the `@`). Without the `@`, Razor passes the literal string "OnDamageClicked" to the DOM and never wires up the C# method.

### Checkpoint

Save. Press Play. Tap the button — the HP number drops by 10 each click, and... well, you can't see the bar yet because there's no styling. The text label updates correctly though.

---

## Step 5: Styling

Add a `<style>` block. CSS lives inside the same `.razor` file:

```html
<style>
    .health-frame {
        position: absolute;
        top: 30px;
        left: 30px;
        width: 320px;
        height: 32px;
        background-color: rgba( 0, 0, 0, 0.5 );
        border: 1px solid #333;
        border-radius: 4px;
        flex-direction: column;
    }

    .health-fill {
        height: 100%;
        background-color: #4caf50;
        transition: width 0.2s ease-out;
    }

    .hp-text {
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        text-align: center;
        font-size: 18px;
        font-family: Poppins;
        color: white;
        text-shadow: 1px 1px 2px black;
    }

    .damage-btn {
        position: absolute;
        top: 80px;
        left: 30px;
        padding: 8px 16px;
        font-size: 16px;
        font-family: Poppins;
        background-color: #c62828;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
    }

    .damage-btn:hover {
        background-color: #e53935;
    }
</style>
```

Some s&box-flavored CSS notes:

- **`flex-direction` is required** for a container to lay out children. Pure block flow doesn't exist in the s&box panel system; everything is flex.
- **`transition`** works for numeric properties. The `width` transition above is what makes the bar smoothly slide instead of snapping.
- **`font-family: Poppins`** uses a font that ships with the engine. Other built-in fonts include `Roboto` and `Inter`. To use a custom font, place a `.fnt` or `.ttf` next to the panel.
- **`rgba()` and `#hex` colors** both work. `hsl()` is also supported.

> **If your styles don't apply**, check that your `<style>` block is at the top level of the file (a sibling of `<root>`), not nested inside it. Nested styles compile but aren't scoped the way you'd expect.

> **If the bar fills but doesn't animate**, the `transition` is being defeated by a parent's `flex-direction: column` resizing. Add `flex-shrink: 0` on `.health-fill` if it gets squeezed.

### Checkpoint

A 320px-wide green bar appears at the top-left, with HP text centered over it and a red button below. Click the button: the bar smoothly shrinks by 10%. Stop and check the file paths in the markup — typos in `class` names won't error, they'll just silently miss any styling.

---

## Step 6: Conditional rendering

Razor supports `@if`, `@for`, and `@foreach` blocks. Let's flash a "DANGER" warning when health drops below 25.

```html
<root>
    <div class="health-frame">
        <div class="health-fill @(LowHealth ? "low" : "")" style="width: @(FillPercent)%"></div>
        <label class="hp-text">HP @Current / @Max</label>
    </div>

    @if ( LowHealth )
    {
        <div class="warning">DANGER</div>
    }

    <button class="damage-btn" onclick="@OnDamageClicked">Take 10</button>
</root>

@code {
    [Property] public float Current { get; set; } = 100f;
    [Property] public float Max { get; set; } = 100f;

    public bool LowHealth => Current < 25f && Current > 0f;
    public int FillPercent => Max > 0
        ? (int)(Current / Max * 100f).Clamp( 0, 100 )
        : 0;

    void OnDamageClicked()
    {
        Current = MathF.Max( 0f, Current - 10f );
    }

    protected override int BuildHash() => System.HashCode.Combine( Current, Max );
}
```

And add to the `<style>`:

```css
.health-fill.low {
    background-color: #ff9800;
    animation: pulse 0.5s infinite alternate;
}

.warning {
    position: absolute;
    top: 80px;
    left: 200px;
    font-size: 24px;
    font-family: Poppins;
    color: #ff5252;
    font-weight: bold;
    animation: pulse 0.6s infinite alternate;
}

@keyframes pulse {
    from { opacity: 1; }
    to { opacity: 0.4; }
}
```

> **Pause and predict:** `LowHealth` is a *computed* property (`=> Current < 25f && Current > 0f`) — it isn't directly listed in `BuildHash`. So why does the panel still re-render at the moment `LowHealth` flips from false to true?
>
> _Continue reading once you've made a guess._

The `@if` block in Razor produces or omits the `<div class="warning">` entirely depending on `LowHealth`. The DOM is rebuilt only when `BuildHash` changes, so flipping `LowHealth` triggers a clean redraw.

The `@(LowHealth ? "low" : "")` inside `class=` is a C# ternary — when true, it adds the `low` modifier class to the fill bar. Compound classes are space-separated.

> **If the warning never appears**, check that `LowHealth` actually flips to true. A common mistake is guarding `Current > 0f` to avoid showing it on death — if you want it visible at 0 too, drop that check.

### Checkpoint

Hammer the damage button until HP < 25. The fill bar turns orange and pulses, and "DANGER" appears next to it. Click again to reach 0; the warning disappears (because of the `Current > 0f` guard).

---

## Step 7: Connect to the rest of the game

So far the HUD has its own `Current` and `Max` and the button drives them. In a real game you want the HUD to *read* from a separate source — the player's health component.

Add a `[Property] public PlayerHealth Source { get; set; }` to your panel:

```csharp
@code {
    [Property] public PlayerHealth Source { get; set; }

    public float Current => Source?.Current ?? 0f;
    public float Max => Source?.MaxHealth ?? 100f;
    public bool LowHealth => Current < 25f && Current > 0f;
    public int FillPercent => Max > 0
        ? (int)(Current / Max * 100f).Clamp( 0, 100 )
        : 0;

    void OnDamageClicked()
    {
        if ( Source is null ) return;
        Source.TakeDamage( 10f );
    }

    protected override int BuildHash() => System.HashCode.Combine( Current, Max );
}
```

Now drag the `Player` (which has the `PlayerHealth` from the [third-person controller tutorial](build-a-third-person-controller.md), or any other component with a `Current` and `MaxHealth`) into the `Source` slot in the Inspector. The HUD reflects the live game state and the button damages the player for real.

> **If the bar reads 0/100 forever**, the `Source` is unset or its `Current` is 0. The first frame after Play often has 0 if your `PlayerHealth.OnStart` runs *after* the panel's first build — the next frame fixes it because `BuildHash` will change once `Source.Current` changes.

---

:::danger UI doesn't show on screen
99% of the time, `ScreenPanel` is missing from the GameObject (or any ancestor of it). `PanelComponent`s render into the nearest root panel. Without one, they exist but draw nowhere.
:::

:::danger Button works once and then doesn't
You probably wrote `[Property] public string Foo` and bound it via `onclick="@Foo"` instead of a method. `onclick` needs a method reference, not a string property.
:::

:::warning HP changes but the screen doesn't
`BuildHash()` is missing the affected fields. Razor only re-renders when the hash changes; if you forgot a field, that field's changes are invisible. Triple-check `System.HashCode.Combine(...)`.
:::

:::warning Style block compiles but produces no effect
The class names don't match (typos), or the markup doesn't have an element where you think it does. Open the in-game **UI Inspector** (`F1` or `View -> UI Inspector` from the Editor) to see the live element tree and computed styles.
:::

## Try extending it

- **Damage numbers.** When the button is clicked, push a new `{ value, time }` entry to a list of recent hits. In markup, `@foreach` over the list and render each as a floating label that fades.
- **Cooldown indicator.** Add a circular cooldown ring around the damage button using SVG (`<svg>` tags work inside Razor markup) and animate the `stroke-dashoffset` based on the remaining cooldown time.
- **World-space HP bar.** Replace `ScreenPanel` with `WorldPanel` and parent the GameObject to the player. The HUD becomes a billboard that hovers above the head. (`WorldPanel`'s pixel size is configurable via its `PanelSize` property.)
- **Multi-player roster.** In a networked game, `Scene.GetAllComponents<PlayerHealth>()` returns every player. `@foreach` them and render a row each. Update `BuildHash` to include the count and a hash of each player's HP.

## Try it yourself

Now add a **"Heal" button** that increases HP by 20 (capped at `Max`). This builds on what you already did with the damage button — same `onclick` binding pattern, same mutation of `Current` — but with a couple of subtle gotchas worth predicting first.

Your task:
- Add a `<button class="heal-btn" onclick="@OnHealClicked">Heal 20</button>` next to the damage button.
- Implement `OnHealClicked` to add 20 to `Current`, clamped at `Max`.
- Style `.heal-btn` so it sits below the damage button and is visibly green.

Predict before you build: when you click Heal while HP is at, say, 10 (below 25, so the DANGER warning is showing), at what value of `Current` will the warning disappear? And will it disappear *the instant the value crosses 25* or only on the next click?

Try it before reading on. Get stuck? Here's a hint:

<details>
<summary>Hint 1</summary>

Markup:

```html
<button class="heal-btn" onclick="@OnHealClicked">Heal 20</button>
```

Method:

```csharp
void OnHealClicked()
{
    Current = MathF.Min( Max, Current + 20f );
}
```

CSS — copy the `.damage-btn` block, change the color, and offset `top` so they don't overlap.

</details>

<details>
<summary>Hint 2 (the answer)</summary>

In your `<root>`:

```html
<button class="heal-btn" onclick="@OnHealClicked">Heal 20</button>
```

In `@code`:

```csharp
void OnHealClicked()
{
    Current = MathF.Min( Max, Current + 20f );
}
```

In `<style>`:

```css
.heal-btn {
    position: absolute;
    top: 130px;
    left: 30px;
    padding: 8px 16px;
    font-size: 16px;
    font-family: Poppins;
    background-color: #2e7d32;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}

.heal-btn:hover {
    background-color: #43a047;
}
```

The prediction answer: the warning disappears the instant `Current` crosses 25, on the same click. The chain is `OnHealClicked` mutates `Current` -> `BuildHash` is now different -> the panel rebuilds -> the `@if ( LowHealth )` block re-evaluates -> the `<div class="warning">` is omitted from the new DOM. All in the same frame. There's no need to manually invalidate anything.

If you'd been mutating `Current` from somewhere `BuildHash` couldn't see (e.g., a parent panel's data that wasn't part of the hash combine), the rebuild wouldn't fire and the warning would stay visible until something else nudged the hash.

</details>

## Next steps

- The full Razor reference, including layout primitives, styling deep-dive, and event lifecycle: [Razor Panels](../systems/ui/razor-panels/index.md).
- For floating-in-3D-space UI, see [WorldPanel](../systems/ui/worldpanel.md).
- To make this UI multiplayer-aware, the [Networking Basics](networking-basics.md) tutorial covers the `[Sync]` and `IsProxy` patterns the HUD needs to display per-player data.
