---
title: "World Panel"
icon: "🌅"
created: 2024-04-11
updated: 2024-04-11
sources:
  - engine/Sandbox.Engine/Scene/Components/UI/WorldPanel.cs
---

# World Panel

> **New to UI?** Reference for the `WorldPanel` component; UI overview is at **[UI](index.md)**.

`WorldPanel` renders any attached UI `PanelComponent`s into the 3D world space, allowing players to view and interact with UI like a physical object.

## Quick Working Example

```csharp
// Create a new GameObject for the world UI
var worldUiObject = new GameObject( true, "Computer Screen UI" );
worldUiObject.WorldPosition = new Vector3( 100, 0, 50 );

// Add the WorldPanel to render UI into 3D space
var worldPanel = worldUiObject.Components.Create<WorldPanel>();
worldPanel.PanelSize = new Vector2( 1920, 1080 );
worldPanel.RenderScale = 1.0f;
worldPanel.LookAtCamera = true;

// Add your custom PanelComponent to display content
// worldUiObject.Components.Create<MyComputerScreenPanel>();
```

### Sizing and Alignment
You define the "resolution" of the panel via `PanelSize`. The `HorizontalAlign` and `VerticalAlign` properties determine where the panel's origin is relative to the GameObject's transform.
```csharp
worldPanel.PanelSize = new Vector2( 800, 600 );
worldPanel.HorizontalAlign = WorldPanel.HAlignment.Center;
worldPanel.VerticalAlign = WorldPanel.VAlignment.Center;
```

### Billboard UI (Look At Camera)
For floating name tags or health bars over enemies, you usually want the UI to always face the player.
```csharp
worldPanel.LookAtCamera = true;
```

## Configuration

| Property | Description |
|---|---|
| `RenderScale` | Multiplier for the visual size of the panel in the 3D world. |
| `LookAtCamera` | If true, the panel will automatically rotate to face the active camera. |
| `PanelSize` | The 2D dimensions (resolution) of the UI canvas (e.g., 1920x1080). |
| `HorizontalAlign` | Origin alignment on the X axis (`Left`, `Center`, `Right`). |
| `VerticalAlign` | Origin alignment on the Y axis (`Top`, `Center`, `Bottom`). |
| `InteractionRange` | How close the player's camera needs to be to click buttons on the panel. |

:::warning UI Text Appears Backwards
If you manually rotate a `WorldPanel` 180 degrees, the UI will be rendered reversed on the back side. Ensure the forward direction (X axis) of the GameObject is pointing towards where you want it to be viewed.
:::

:::danger Cannot Click Buttons
If your UI looks fine but you can't click anything, check the `InteractionRange` property. If the player is further away than this distance, interactions will be ignored.
:::

## Related Pages
* [UI Overview](index.md)
* [Screen Panel](screenpanel.md)
