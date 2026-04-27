---
title: "Menu Modals"
icon: "🔔"
sources:
  - game/addons/menu/Code/Modals/
created: 2026-04-27
updated: 2026-04-27
---

# Menu Modals

The Menu Addon uses a centralized modal system to display popups, dialogs, and overlays within the main menu environment. 

This system allows for standardized dialogs like friend lists, package selections, and confirmations.

## Quick Start Example

When you want to create a new custom modal, inherit from `BaseModal` and use Razor for the UI definition:

```razor
@using Sandbox
@namespace MenuProject.Modals
@inherits MenuProject.Modals.BaseModal

<root class="my-custom-modal">
    <div class="inner">
        <div class="header">
            <h2>Custom Modal</h2>
        </div>
        <div class="content">
            <p>This is a custom modal popup.</p>
            <button class="primary" onclick=@(() => CloseModal(true))>Accept</button>
            <button class="secondary" onclick=@(() => CloseModal(false))>Cancel</button>
        </div>
    </div>
</root>
```

You can then spawn this modal dynamically within the menu's UI hierarchy.

## Common Patterns

1. **Options Structs:** Many complex modals pass an "Options" struct into their constructor. This allows the invoker to configure the modal's behavior (e.g., `FriendsListModalOptions` defining whether to show offline friends).
2. **State Updates:** Modals often listen to engine events (like `[Event("friend.change")]`) and call `StateHasChanged()` to re-render the Razor UI when underlying data changes.
3. **Yielding for Input:** While not natively async, you can wrap the instantiation and `OnClosed` callback in a TaskCompletionSource if you want to `await` the user's choice in an asynchronous method.

## Built-In Modals

The menu addon ships with several ready-to-use modals:
- `FriendsListModal`: Displays the player's Steam friends list, categorized by online/playing status.
- `PackageSelectionModal`: A modal to browse and select packages (addons/games) from sbox.game.
- `ServerListModal`: A modal overlay showing active dedicated servers for a specific game.
- `ReviewModal`: A popup to leave a star rating and review for a played package.

## Troubleshooting

:::warning Common Gotchas
- **Modal doesn't close on background click:** Ensure you didn't accidentally override the base constructor in C# without calling `base()`, or accidentally remove the background panel in your Razor template if you overrode the entire DOM structure.
- **UI doesn't update:** If external data changes (like a friend going offline) while the modal is open, ensure you are listening for the appropriate Event and calling `StateHasChanged()`.
:::

## Related Pages
- [Menu Architecture](./menu-architecture.md)
- [Menu UI](./menu-ui-components.md)

