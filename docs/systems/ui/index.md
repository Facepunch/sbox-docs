---
title: "UI"
icon: "🖥️"
created: 2024-09-24
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Systems/UI/Panel/Panel.cs
---

# UI

s&box's UI is built on **Panels** — a tree of HTML-like elements styled with CSS and laid out with flexbox. You can write panels two ways: directly in C# (`new Panel { Parent = this }`) or in `.razor` files that look like HTML markup with C# bindings.

Most game devs use Razor. It's faster to iterate on, the markup is closer to what you'll actually see, and the data-binding is one-line: type `@Player.Health` in your markup and the panel re-renders when `Health` changes.

## Two flavours of UI surface

UI lives on a `PanelComponent`. The `PanelComponent` lives on a GameObject, which has either a `ScreenPanel` or a `WorldPanel` to decide where it renders:

| Component | Where the UI shows up |
|---|---|
| `ScreenPanel` | Drawn flat over the camera's output — HUDs, menus |
| `WorldPanel` | Drawn in 3D space, attached to a position in the world — signs, name tags, holograms |

For a screen HUD, you'd:

1. Have a GameObject with both `ScreenPanel` and your `PanelComponent`-derived class.
2. Your `PanelComponent` class has a matching `.razor` file that defines the markup.
3. The engine builds the panel tree on first frame and ticks it.

## A minimal Razor HUD

`Hud.cs`:

```csharp
using Sandbox;

public sealed class Hud : PanelComponent
{
    [Property] public PlayerStats Player { get; set; }

    protected override int BuildHash() => System.HashCode.Combine( Player?.Health );
}
```

`Hud.razor`:

```razor
@using Sandbox;
@inherits PanelComponent

<root>
    <div class="health-bar">
        <div class="fill" style="width: @(Player.Health)%"></div>
    </div>
</root>

<style>
    .health-bar { width: 200px; height: 20px; background: rgba(0,0,0,0.4); }
    .fill       { height: 100%; background: red; transition: width 0.2s; }
</style>
```

`BuildHash` returns a value that changes whenever the panel should re-render. The engine compares the hash to detect changes; if it changes, the panel rebuilds.

## C# panels (without Razor)

If you'd rather build panels in code:

```csharp
public class MyPanel : Panel
{
    public Label Time { get; set; }

    public MyPanel()
    {
        Time = new Label { Parent = this };
    }

    public override void Tick()
    {
        Time.Text = $"{Time.Now}";
    }
}
```

Then either drop it into a `PanelComponent` from `OnTreeFirstBuilt`, or reference it from a parent Razor template via `<MyPanel />`.

## Where to read more

- [Razor Panels](razor-panels/index.md) — full Razor syntax, `@if`, loops, components-within-components, the `[Bind]` attribute
- [Styling Panels](styling-panels/index.md) — supported CSS properties, transitions, animations
- [Screen Panel](screenpanel.md) — HUD panels, scaling modes, ordering
- [World Panel](worldpanel.md) — UI in 3D space
- [World Input](worldinput.md) — handling clicks on a `WorldPanel`
- [Scene Panel](scenepanel.md) — render a separate 3D scene into a UI element (loadout previews, character viewer)
- [Virtual Grid](virtualgrid.md) — efficient lists with thousands of items (inventory, leaderboards)
- [HudPainter](hudpainter.md) — immediate-mode 2D drawing for crosshairs, gauges, custom widgets
- [Localization](localization.md) — translating UI text

For the recipe-level basics: [Build a User Interface (tutorial)](../../tutorials/build-a-ui.md), [Create a UI Health Bar (recipe)](../../how-to/health-bar.md).

## Things that catch people out

**UI doesn't render.** The most common cause: your `PanelComponent` is on a GameObject that doesn't also have a `ScreenPanel` (or `WorldPanel`). The component pair is required.

**UI looks stretched at certain resolutions.** `ScreenPanel` defaults to scaling against a 1080p target height. If your target is desktop-resolution-native, change the `ScreenPanel.Scale` setting.

**Panel doesn't update when state changes.** Either your `BuildHash` doesn't include the value that changed, or the value you're displaying lives outside the component and Razor can't see it changed. The simplest fix: include all the values your markup reads in `BuildHash`.

**Razor file changes don't hotload visually.** Save the `.razor` and the matching `.cs` will recompile, but the *running panel instance* keeps its old generated tree. Triggering a property change to invalidate the hash usually rebuilds it; otherwise restart the scene.

## Inspect at runtime

Press **F1** in-game (default binding) to open the UI Inspector — it lets you click any panel on screen and see its element tree, computed styles, and source. Indispensable for debugging layout.
