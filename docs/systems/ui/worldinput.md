---
title: "World Input"
icon: "🔄"
sources:
  - engine/Sandbox.Engine/Scene/Components/UI/WorldInput.cs
updated: 2026-04-25
created: 2026-04-27
---

# World Input

> **New to UI?** Reference for the `WorldInput` component; UI overview is at **[UI](index.md)**.

The `WorldInput` component acts as a router for world input, allowing players to interact with `WorldPanel` components using mouse raycasting or VR controllers.

## Quick Working Example

```csharp
using Sandbox;

public sealed class PlayerCamera : Component
{
	protected override void OnStart()
	{
		// Add WorldInput to the camera so it can raycast into the world
		var worldInput = Components.Create<WorldInput>();
		worldInput.LeftMouseAction = "attack1";
		worldInput.RightMouseAction = "attack2";
	}
}
```

### Placing on the Camera

The most common setup for a first-person or third-person game is to attach the `WorldInput` component directly to the same GameObject that has your `CameraComponent`. If the mouse is active, it uses a ray generated from the mouse cursor. Otherwise, it simply projects a ray straight forward from the GameObject.

### Reading Hovered Panels

You can check if the player is currently looking at a World Panel by querying the `Hovered` property.

```csharp
var worldInput = Components.Get<WorldInput>();
if ( worldInput.Hovered != null )
{
	Log.Info( "Player is looking at a UI panel!" );
}
```

## Configuration

| Property | Description |
|:---|:---|
| `LeftMouseAction` | The Input Action name (e.g. `Attack1`) that triggers a left-click on the UI. |
| `RightMouseAction` | The Input Action name (e.g. `Attack2`) that triggers a right-click on the UI. |
| `VRHandSource` | Which VR hand to use for input (`Left` or `Right`). Triggers are treated as left clicks, and the joystick as scrolling. |
| `Hovered` | (Read-only) The `Panel` that is currently hovered by this input. |

## Troubleshooting

:::danger "I can't click anything on my World Panel!"
First, ensure that a `WorldInput` component is active in your scene (usually on the Camera). Second, check the `InteractionRange` property on the `WorldPanel` itself—if the player is too far away, clicks will be ignored.
:::

:::warning "WorldInput isn't detecting my custom actions!"
Make sure that `LeftMouseAction` and `RightMouseAction` exactly match the action names you have defined in your Project Settings.
:::

## Related Pages
* [World Panel](worldpanel.md)
* [Input Overview](../input/index.md)
