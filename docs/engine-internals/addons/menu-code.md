---
title: "Menu Addon Code Architecture"
sources:
  - game/addons/menu/Code/MenuSystem.cs
  - game/addons/menu/Code/ModalSystem.cs
  - game/addons/menu/Code/MenuUI/
created: 2026-04-27
updated: 2026-04-27
---

# Menu Addon Code Architecture

The Main Menu addon (`game/addons/menu/Code/`) provides the frontend UI that users interact with when launching s&box. This codebase manages the server browser, avatar editor, modals, and overarching menu states.

## Quick Working Example

You can trigger modals or menu functions programmatically if you're building upon the existing menu structure. 

```csharp
using Sandbox;
using Sandbox.Menu;

public sealed class MenuLauncher : Component
{
    protected override void OnStart()
    {
        // Example: Push a new standard modal to the menu system
        ModalSystem.Instance.Settings();
    }
}
```

1. **Building Custom Modals:** To create a new popup, you define a new Razor component in the `Modals/` folder and call it via a method added to `ModalSystem.Instance`.
2. **Avatar Integration:** The `AvatarEditor/` directory contains components that render and manipulate the 3D Citizen model directly within the 2D UI layout using scene panels.

- **Background Sleeping:** To preserve performance for active games, the menu aggressively sleeps when hidden. Any loops or `OnUpdate` behaviors written in the menu code will pause or tick very slowly when a user is in a game.

## Troubleshooting

:::warning Common Gotchas
- **State Desync:** If you modify state in a child Razor component but don't call `StateHasChanged()`, the UI won't reflect the update.
- **Leaked Modals:** Make sure you use the `ModalSystem` properly rather than instantiating raw modal panels, or they may fail to close when the user hits Escape.
:::

## Related Pages
- [Main Menu Addon](index.md)

