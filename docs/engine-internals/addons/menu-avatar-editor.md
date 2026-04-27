---
title: "Menu Avatar Editor"
icon: "🙂"
sources:
  - game/addons/menu/Code/AvatarEditor/
created: 2026-04-27
updated: 2026-04-27
---

# Menu Avatar Editor

The Menu Avatar Editor provides the UI and background rendering logic for players to customize their global Citizen character appearance directly from the main menu.

## Quick Start Example

If you are modifying the menu addon and want to interact with the currently edited avatar state:

```csharp
using Sandbox;
using Sandbox.UI;

public sealed class AvatarDebug : PanelComponent
{
    protected override void OnUpdate()
    {
        // Example: Check if the user is currently editing their avatar
        if ( AvatarEditManager.Instance.IsValid() && AvatarEditManager.Instance.IsEditing )
        {
            Log.Info( "User is currently in the Avatar Editor screen." );
        }
    }
}
```

## Common Patterns

1. **Scene Integration:** The Menu addon renders a 3D scene behind its 2D Razor UI. The `AvatarEditManager` communicates with the `AvatarBackgroundRig` in this scene to update the visible character mesh whenever a new clothing item is selected.
2. **Backend Syncing:** When a user finalizes their outfit and clicks "Save," the `AvatarEditManager` serializes the `ClothingContainer` state and syncs it with the sbox.game backend, ensuring the outfit persists across different games.

## Troubleshooting

:::warning Common Gotchas
- **Missing Items:** The Avatar UI populates its list by querying the backend for authorized items and scanning the local `citizen` addon. If custom clothing from an addon isn't appearing, verify that it is properly tagged as a `Clothing` resource and is not restricted.
- **Preview Not Updating:** If you modify `AvatarEditManager` logic but the 3D model doesn't change, ensure you are calling the appropriate refresh or rebuild methods on the underlying `ClothingContainer` attached to the background rig.
:::

## Related Pages
- [Menu Architecture](./menu-architecture.md)
- [Citizen Addon](../../build-games/addons/citizen/index.md)

