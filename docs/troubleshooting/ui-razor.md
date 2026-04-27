---
title: UI & Razor
icon: "🌐"
created: 2026-04-25
updated: 2026-04-25
---

# UI & Razor

Panels not rendering, Razor not updating, styles not applying, world panels not clickable.

## UI not appearing

### "My UI isn't rendering on screen"

A `PanelComponent` needs a `ScreenPanel` (or `WorldPanel`) on the same GameObject — that's what tells the engine *where* to draw the UI.

- **Fix:** add a `ScreenPanel` component alongside your custom `PanelComponent`.
- **Check:** `Opacity > 0` on the ScreenPanel.
- **Check:** the GameObject is enabled.

**Deep dive:** [UI Index](../systems/ui/index.md), [Screen Panel](../systems/ui/screenpanel.md).

### "I can't add my Panel to a GameObject"

Your `.razor` file inherits from `Panel` instead of `PanelComponent`.

- **Fix:** at the top of the `.razor`, write `@inherits PanelComponent`. Only `PanelComponent` is attachable to GameObjects.

**Deep dive:** [Razor Panels](../systems/ui/razor-panels/index.md).

### "I can't click anything on my World Panel"

Either no `WorldInput` component is active, or the player is too far for `InteractionRange`.

- **Check:** a `WorldInput` component exists in the scene (usually on the Camera).
- **Check:** `InteractionRange` on the WorldPanel is large enough.

**Deep dive:** [World Input](../systems/ui/worldinput.md), [World Panel](../systems/ui/worldpanel.md).

### "My ScenePanel is completely black or empty"

The Scene you're rendering doesn't have a Camera or has no lights.

- **Fix:** the Scene assigned to `RenderScene` needs at least a `CameraComponent` and lighting.
- **Note:** if `RenderOnce = true`, only the first frame renders. Set false for animations.

**Deep dive:** [Scene Panel](../systems/ui/scenepanel.md).

## UI not updating

### "My UI doesn't update when my variables change"

Razor only redraws when `BuildHash()` returns a different value. If you didn't include the variable in the hash, the renderer thinks nothing changed.

- **Fix:** override `protected override int BuildHash() => System.HashCode.Combine( Health, Score, ... );` and include every value the UI displays.

**Deep dive:** [Razor Panels](../systems/ui/razor-panels/index.md), [Health Bar Tutorial](../how-to/health-bar.md).

### "The button clicks, but the displayed value never changes"

Same root cause: `BuildHash()` doesn't include the value.

- **Fix:** add the value to `HashCode.Combine( ... )`.

### "My SCSS styles aren't updating"

Naming convention violated. `MyComponent.razor` must pair with **`MyComponent.razor.scss`** — not `mycomponent.scss`, not `MyComponent.scss`.

- **Fix:** rename the SCSS file to match exactly, including the `.razor.scss` suffix.

**Deep dive:** [Styling Panels](../systems/ui/styling-panels/index.md).

### "Can I use float, CSS Grid, or display: block?"

s&box uses Flexbox-based layout. `float`, CSS Grid, and traditional block layout are not supported.

- **Fix:** use `flex-direction`, `justify-content`, `align-items`, `flex: 1 1 auto`.

## Razor compilation

### "I'm getting compile errors pointing to invisible code in my .razor"

The Razor compiler generates C# behind the scenes. Errors point to the *generated* line, not your `.razor` source.

- **Fix:** check the auto-generated `.cs` file (usually under `obj/`). The error column will help you back-track to the offending Razor line.
- **Common cause:** unclosed `@{ ... }` block, missing `@` on a method call, or an `@inherits` mismatch.

**Deep dive:** [Razor UI Architecture](../systems/ui/razor/architecture.md), [Sandbox.Razor](../engine-internals/sandbox-razor.md).

### "Component tag is not recognized"

Namespace mismatch.

- **Fix:** add `@using YourNamespace` at the top, or place both components in the same namespace.

**Deep dive:** [Razor Components](../systems/ui/razor-panels/razor-components.md).

### "Razor folder-based namespacing isn't working"

The auto-namespace skips folders with **spaces**, **hyphens (-)**, or that **start with a number**.

- **Fix:** rename folders to use underscores or PascalCase.

## Style and layout

### "I'm getting errors setting Style.Width on my PanelComponent"

`PanelComponent` is a wrapper, not a Panel itself. You need the inner panel.

- **Fix:** `Panel.Style.Width = Length.Pixels( 100 );` — note the `Panel.` prefix to reach the inner panel.

### "UI looks stretched or tiny"

`AutoScreenScale` and `ScaleStrategy` need tuning for your target resolution.

- **Fix:** for crisp 1:1 pixels, set `AutoScreenScale = false` and manage `Scale` manually.
- **Fix:** for design-resolution scaling, set the strategy to `Width` or `Height` based on your design width/height.

**Deep dive:** [Screen Panel](../systems/ui/screenpanel.md).

### "Health bar fill draws outside its container"

CSS `overflow` is missing on the container.

- **Fix:** `overflow: hidden` on the container element so values >100% don't draw past the bounds.

**Deep dive:** [Health Bar](../how-to/health-bar.md).

### "Gaps aren't appearing between my VirtualGrid items"

`VirtualGrid` supports CSS gap properties — apply them to the grid, not to the items.

- **Fix:** `row-gap`, `column-gap`, or shorthand `gap` on the grid container's CSS.

### "VirtualGrid is squished or doesn't show"

Grid needs an explicit size to compute how many items fit in view.

- **Fix:** set `width` and `height` (or flex sizing) on the grid in CSS.

**Deep dive:** [Virtual Grid](../systems/ui/virtualgrid.md).

## Drag, drop, scroll

### "My drag is being eaten by the parent scroll container"

Scroll containers grab pointer events first.

- **Fix:** in your draggable's `OnDragStart`, call `e.StopPropagation()`.

**Deep dive:** [UI Drag and Drop](../how-to/ui-drag-and-drop.md).

## Localization

### "My text shows as `#menu.title` instead of the translation"

The token isn't loaded for the current language.

- **Check:** the JSON exists for that language code (e.g., `localization/en.json`).
- **Check:** the JSON is well-formed (a single missing comma kills the whole file).
- **Check:** the `language.json` map registers your language.

**Deep dive:** [Localization](../systems/ui/localization.md).

### "Variables in my translated text aren't being replaced"

`Language.GetPhrase()` does case-sensitive key matching for substitutions.

- **Fix:** `{playerName}` in the JSON must match `{ "playerName", "Bob" }` in the code dictionary, including case.

## World Panel quirks

### "World panel UI text appears backward"

You rotated the WorldPanel 180°, so the back of the panel faces the viewer.

- **Fix:** make the GameObject's forward (X axis) point towards where you want it to be viewed.

**Deep dive:** [World Panel](../systems/ui/worldpanel.md).

## Related Pages

- [Scene & Components](./scene-and-components.md)
- [Editor](./editor.md)
- [Hotload & Compile](./hotload-and-compile.md)
