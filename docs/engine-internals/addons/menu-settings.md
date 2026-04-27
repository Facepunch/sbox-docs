---
title: "Menu Settings"
icon: "⚙️"
sources:
  - game/addons/menu/Code/Modals/Settings/
created: 2026-04-27
updated: 2026-04-27
---

# Menu Settings

The Menu Settings system manages the options screen visible from the main menu, handling everything from video resolution to input bindings and audio volumes. It serves as the primary interface between the player and the engine's global configuration.

## Quick Start Example

As a contributor adding a new game setting, you'll generally interact with the Razor pages that define the settings UI.

```razor
@using Sandbox
@using MenuProject.Modals.Settings

<!-- Adding a new toggle to GameSettings.razor -->
<div class="form-row">
    <label>Show Framerate</label>
    <div class="control">
        <SwitchControl Value:bind=@DeveloperSettings.ShowFps />
    </div>
</div>
```

## Page Structure

The Settings Modal is broken down into thematic Razor pages:

- **VideoSettings:** Handles display resolution, VSync, framerate limits, and overall graphics quality tiers.
- **AudioSettings:** Manages master, music, and sound effect volumes.
- **InputSettings:** Handles mouse sensitivity, controller deadzones, and key bindings.
- **GameSettings:** General gameplay preferences, such as field of view (FOV) and UI scaling.
- **DeveloperSettings:** Toggles for debug overlays, performance metrics (FPS), and developer console access.

## Common Patterns

1. **Two-Way Binding:** Nearly all settings rely on Razor's `:bind` syntax. This eliminates boilerplate event handlers; the UI control updates the engine property directly.
2. **Immediate Application vs Apply Button:** In modern s&box, most settings apply immediately upon changing the slider or toggle. There is generally no "Apply" button required, except for significant display changes that might require a confirmation prompt.

## Troubleshooting

:::warning Common Gotchas
- **Setting doesn't persist:** Ensure the underlying engine property is actually decorated to save to config. The Razor UI only changes the property in memory; if the engine doesn't serialize it, it will reset on restart.
- **UI doesn't reflect external changes:** If a setting is changed via the developer console instead of the menu UI, the Razor page must be rebuilt or hooked to an event to reflect the new value visually.
:::

## Related Pages
- [Menu Modals](menu-modals.md)
- [Menu Architecture](./menu-architecture.md)

