---
title: Base Addon
sources:
  - game/addons/base
created: 2026-04-27
updated: 2026-04-27
---

# Base Addon

The `base` addon, located at `game/addons/base/`, is the fundamental dependency for almost all s&box projects. It provides the essential classes, UI controls, and utility systems needed to build a game on top of the engine.

## Quick Working Example

When you create a new game project, it automatically includes `base` as a dependency. You can immediately use its UI controls in your Razor templates.

```html
@using Sandbox;
@using Sandbox.UI;
@inherits PanelComponent

<root>
    <div class="menu">
        <label>Settings</label>
        <!-- The Checkbox class is provided by the Base addon -->
        <Checkbox @bind-Value="EnableSound" />
    </div>
</root>

@code {
    public bool EnableSound { get; set; } = true;
    
    protected override int BuildHash() => System.HashCode.Combine( EnableSound );
}
```

## What's Included?

The `game/addons/base/code/` directory is massive, but it generally breaks down into these key areas:
*   **UI Controls (`UI/Controls/`):** Dozens of standard Razor components like `Button`, `TextEntry`, `Slider`, `ColorPicker`, and `Checkbox`.
*   **Form Layouts (`UI/Form/`):** Helpers for building settings menus and data-entry forms.
*   **Core Components (`Components/`):** Base logic for common networking patterns and standard utilities.
*   **Citizen Specifics (`Components/Citizen/`):** Helpers specifically tailored for interacting with the standard Citizen character model (e.g., managing clothing and animation states).

## Usage Patterns

1. **Inheritance & Extension:** Rather than writing a text input field from scratch, inherit from or compose your UI using `TextEntry` from the base addon. You can restyle it using standard CSS without changing its underlying C# logic.
2. **Citizen Integration:** If your game uses the default Citizen models, the `CitizenAnimationHelper` component in the base addon handles the complex animation graph state machines for you. Just pass it your velocity and look direction.

## Performance

When using complex base UI controls (like `ColorPicker` or large `ControlSheet`s), be aware that they generate many DOM nodes. Instantiating hundreds of these controls simultaneously will trigger a heavy layout recalculation pass. Use virtualization (only rendering what is on screen) if you are building massive UI lists.

- **Asset Overrides:** If you place a file with the exact same path and name as an asset in the `base` addon within your own addon (e.g., `materials/dev/dev_measuregeneric01.vmat`), your addon's version will *override* the base version. This is incredibly powerful for reskinning standard tools globally but can cause unexpected visual bugs if done accidentally.

## Troubleshooting

:::warning Common Gotchas
- **"Type 'Button' could not be found"**
  If the compiler complains about missing standard UI components, your project is likely missing the `facepunch.base` dependency in its `.sbproj`, or you are missing `using Sandbox.UI;` in your C# file.
- **Editing Base Files:**
  **Never edit files directly inside `game/addons/base/`.** If you update the engine via Steam, your changes will be overwritten and lost. If you need to change how a base script works, copy it to your own project and modify it there.
:::

## Exploring the Source

If you want to understand how to build robust, reusable Razor components, the `game/addons/base/code/UI/Controls/` directory is the best place to learn. The engine developers built the entire s&box Editor UI using these exact same patterns.

## Related Pages
- [Runtime Layout](../runtime-layout/index.md)
- [Citizen Addon](../../build-games/addons/citizen/index.md)

