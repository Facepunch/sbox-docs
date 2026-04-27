---
title: "Screen Panel"
icon: "🖥️"
created: 2024-04-11
updated: 2024-04-11
sources:
  - engine/Sandbox.Engine/Scene/Components/UI/ScreenPanel.cs
---

# Screen Panel

> **New to UI?** Reference for the `ScreenPanel` component; UI overview is at **[UI](index.md)**.

`ScreenPanel` renders any attached UI `PanelComponent`s directly to the user's screen in 2D space.

## Quick Working Example

```csharp
// Create a new GameObject to hold our UI
var uiObject = new GameObject( true, "Screen UI" );

// Add the ScreenPanel, which acts as the root renderer for the screen
var screenPanel = uiObject.Components.Create<ScreenPanel>();
screenPanel.ZIndex = 100;
screenPanel.Scale = 1.0f;

// Now add your custom PanelComponent to the same GameObject
// uiObject.Components.Create<MyCustomPanel>();
```

### Setting Z-Index for Layering
If you have multiple ScreenPanels (e.g., a HUD and a Pause Menu), you can control which draws on top using `ZIndex`. Higher numbers draw on top of lower numbers.
```csharp
screenPanel.ZIndex = 200; // Draws above ZIndex 100
```

### Targeting Specific Cameras
In split-screen or multi-camera setups, you can make a `ScreenPanel` only render for a specific camera.
```csharp
var mainCamera = Scene.GetAll<CameraComponent>().FirstOrDefault();
screenPanel.TargetCamera = mainCamera;
```

## Configuration

| Property | Description |
|---|---|
| `Opacity` | Master opacity for everything rendered in this panel (0.0 to 1.0). |
| `Scale` | Multiplier for the size of the UI elements. |
| `AutoScreenScale` | If true, automatically adjusts scale based on `ScaleStrategy`. |
| `ScaleStrategy` | `ConsistentHeight` (assumes 1080p target) or `FollowDesktopScaling`. |
| `ZIndex` | Sort order for multiple ScreenPanels. Higher draws on top. |
| `TargetCamera` | The specific `CameraComponent` to render to. If null, renders to the main view. |

:::danger UI Not Showing Up
Ensure your custom `PanelComponent` is attached to the *same* GameObject as the `ScreenPanel` (or a child of it). Also verify `Opacity` is greater than 0 and the UI is active.
:::

:::warning UI Looks Stretched or Tiny
Check your `AutoScreenScale` and `ScaleStrategy` settings. If you want crisp 1:1 pixel rendering, try setting `AutoScreenScale` to `false` and manually managing the `Scale`.
:::

## Related Pages
* [UI Overview](index.md)
* [World Panel](worldpanel.md)
