---
title: "Sandbox.Menu"
icon: "☰"
sources:
  - engine/Sandbox.Menu/MenuDll.cs
  - engine/Sandbox.Menu/MenuUtility.cs
  - engine/Sandbox.Menu/MenuScene.cs
  - engine/Sandbox.Menu/Extensions/PackageExtensions.cs
  - engine/Sandbox.Menu/Console/MenuConVar.cs
updated: 2026-04-25
created: 2026-04-27
---

# Sandbox.Menu

The `Sandbox.Menu` namespace provides the engine-level API for interacting with the main menu and overlay systems.

## Extensibility

This namespace exposes static methods and classes that the Menu addon uses. If you are modifying the engine to support a new type of backend service or a new platform, you would expose those hooks here so the C# UI can access them.

## MenuDll Bootstrapping

`MenuDll` implements the internal `IMenuDll` interface, which acts as the core entry point for the AppSystem to initialize and communicate with the menu context. The `IMenuDll` interface provides critical lifecycle hooks for the menu environment, including methods like `LateTick()` (which pauses the menu on escape key presses), `HasOverlayMouseInput()` (which reports if the UI is intercepting input), and explicitly manages the destruction and recreation of the entire `MenuScene` internally via standard `OnGameEntered()` / `Reset()` callbacks.

When the native engine is ready, it calls `MenuDll.Create()` to spin up the managed instance, followed by `Bootstrap()`. This bootstrap sequence:
- Initializes the `GlobalContext` specifically for the menu's isolated assembly scope.
- Configures the `Game` host for menu operations.
- Initializes input contexts and Steam integration (if not running in editor).
- Creates an `ExpirableSynchronizationContext` (`MenuDll.AsyncContext`) to ensure that menu-related asynchronous tasks execute safely within the same thread context as the menu itself.

## MenuUtility

The core of menu interaction revolves around `MenuUtility`, a static class which bridges game UI logic to the low-level functions such as accessing friends list, saving avatars, posting reviews, or managing Steam lobbys.

### Common `MenuUtility` Interactions
- **Steam/Friends Access:** Properties like `Friends` provide a list of steam friends. You can call `JoinFriendGame( friend )` to connect to a lobby they are in.
- **Connection Management:** Use `Connect( lobbyId )` to connect to a server.
- **Avatar/Storage:** Methods like `SaveAvatar( clothingContainer, isActive, slot )` save the user's avatar setups.
- **Feed and Notifications:** Hooks into backend services allow the menu to display the user feed (`GetPlayerFeed`), notifications (`GetNotifications`), and achievements.

### StoragePublish
Nested alongside `MenuUtility` is `StoragePublish`, a helper class used internally to handle the submission of user generated content (UGC). It provides a unified API to bundle and upload an addon to the backend by automatically populating `Ugc.UgcPublisher`.

When creating a `StoragePublish` instance, you can define core package metadata like `Title`, `Description`, `Visibility`, `Tags`, and a `Thumbnail` bitmap. It also allows setting custom `KeyValues` for extra backend metadata and assigning a virtual `FileSystem` that dictates exactly which directories and files are packed and uploaded to Steam UGC. If you specify an existing `PublishedFileId`, the `Submit()` call will update the existing package rather than creating a new one.

```csharp
var publisher = new StoragePublish
{
    Title = "My Addon",
    Description = "A cool addon.",
    Tags = new HashSet<string> { "addon", "fun" },
    Visibility = Storage.Visibility.Public,
    FileSystem = myVirtualFileSystem
};
await publisher.Submit();
```

## MenuScene
`MenuScene` is responsible for initializing and ticking the standalone scene that lives behind the UI of the main menu. When `MenuScene.Tick()` is invoked, it explicitly ticks the background scene but *only if* the main menu is visible (`Game.IsMainMenuVisible`).

:::warning Main Menu Tick Rate
Because the main menu scene only processes game ticks while the overlay is visible, any custom physics or complex systems running in the main menu scene will pause as soon as the user enters a game or hides the menu.
:::

## Package Extensions
The `Sandbox.Menu` namespace also provides several helpful extensions tailored to the menu addon logic to deal with `Package` elements:
- `SetFavouriteAsync( state )`: Favourites or unfavourites the package using backend endpoints.
- `SetVoteAsync( up )`: Sets the user's up/down vote for a package.
- `OpenModal()`: Depending on the package's `TypeName` ("game", "map", etc.), this will automatically open the appropriate UI Modal overlay by calling the internal `Game.Overlay.ShowGameModal( ... )` methods.

## Menu Console Commands & Variables
If you need to declare ConVars or ConCmds that are restricted to the context of the main menu, use the specialized attributes:
- `[MenuConVar( "my_var" )]`
- `[MenuConCmd( "my_cmd" )]`

These operate identically to the standard `ConVar` / `ConCmd` attributes but implicitly set their `Context` to `"menu"`.

## Related Pages
- [Menu Addon](addons/index.md)
