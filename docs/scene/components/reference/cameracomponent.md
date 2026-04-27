---
title: "Camera Component"
icon: "📹"
sources:
  - engine/Sandbox.Engine/Scene/Components/Camera/CameraComponent.cs
  - engine/Sandbox.Engine/Scene/Components/Camera/CameraComponent.PostProcess.cs
updated: 2026-04-25
created: 2026-04-27
---

# Camera Component

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `CameraComponent` component.

Every scene needs at least one `CameraComponent`. It acts as the viewpoint for your game, capturing the 3D world and rendering it to the screen or to a texture.

## Quick Working Example

```csharp
public class CameraSetup : Component
{
	protected override void OnStart()
	{
		// Access the main camera in the current scene
		var camera = Scene.Camera;

		if ( camera != null )
		{
			// Adjust the field of view
			camera.FieldOfView = 90f;

			// Change the background color
			camera.BackgroundColor = Color.Black;
		}
	}
}
```

### Converting Screen to World

A common task is figuring out what the player is pointing at with their mouse. You can generate a `Ray` from a pixel position on the screen into the 3D world.

```csharp
// Access the main camera
var camera = Scene.Camera;

// Get a ray from the center of the screen
var ray = camera.ScreenPixelToRay( camera.ScreenRect.Center );

// Trace that ray to see what it hits
var tr = Scene.Trace.Ray( ray, 5000f ).Run();
if ( tr.Hit )
{
	Log.Info( $"Looking at: {tr.GameObject.Name}" );
}
```

### Converting World to Screen

Conversely, you might want to know where a 3D object appears on the 2D screen (e.g., to draw a name tag over a player's head).

```csharp
var camera = Scene.Camera;
var targetPos = GameObject.WorldPosition;

// Get the pixel coordinates of the 3D position
var screenPos = camera.PointToScreenPixels( targetPos, out bool isBehind );

if ( !isBehind )
{
	// Draw UI element at screenPos
}
```

### Rendering to a Texture

Sometimes you want a camera to render its view into a texture instead of the main screen (e.g., for security cameras, mirrors, or UI elements). This can be achieved by assigning a texture to the camera's `RenderTarget` or `RenderTexture`.

```csharp
public class SecurityCamera : Component
{
	[Property] public CameraComponent Camera { get; set; }
	[Property] public Texture RenderTarget { get; set; }

	protected override void OnStart()
	{
		if ( Camera != null && RenderTarget != null )
		{
			// The camera will now render to this texture instead of the screen
			Camera.RenderTarget = RenderTarget;
		}
	}
}
```

## Configuration

| Property | Description |
|:---|:---|
| `IsMainCamera` | Returns true if this is the main game camera. The engine automatically sets the main camera based on priority. |
| `ClearFlags` | Dictates what parts of the render target get cleared before drawing (Color, Depth, Stencil). |
| `BackgroundColor` | The color drawn behind everything else if there's no 2D Sky in the scene. |
| `FieldOfView` | The field of view in degrees. Larger values show more of the world. |
| `FovAxis` | The axis to use for the field of view (`Horizontal` or `Vertical`). |
| `ZNear` / `ZFar` | The near and far clipping planes. Objects outside this range aren't rendered. A good `ZNear` is 5-10, `ZFar` is 1000-10000. |
| `Priority` | Dictates which camera gets rendered on top of another. Higher means it'll be rendered on top. |
| `Orthographic` | If true, renders without perspective (objects don't get smaller in the distance). |
| `OrthographicHeight` | The height of the orthographic view when `Orthographic` is true. |
| `Viewport` | The size of the camera represented on the screen, normalized between 0 and 1. |
| `TargetEye` | The HMD eye that this camera is targeting for VR (`None`, `LeftEye`, `RightEye`, `Both`). |
| `RenderTags` | Only objects with these tags will be rendered. |
| `RenderExcludeTags` | Objects with these tags will be hidden. |
| `RenderTexture` | If specified, this camera will render to this `RenderTexture` instead of the screen. |
| `RenderTarget` | If specified, this camera will render to this native `Texture` instead of the screen. |
| `EnablePostProcessing`| Toggles whether post-processing effects are applied to this camera. |
| `PostProcessAnchor` | If set, post process volumes are triggered from this `GameObject`'s position, instead of the camera's position. |

## Troubleshooting

:::danger "My screen is totally black!"
Make sure you have a `CameraComponent` in your scene, and that the `GameObject` it is attached to is active. If you are instantiating scenes or objects dynamically, ensure the camera isn't getting destroyed or disabled. Also, ensure `IsMainCamera` is true for your primary camera.
:::

:::warning "I see z-fighting or flickering textures!"
Check your `ZNear` property. Setting `ZNear` too close to 0 (like 0.1 or 0.01) reduces depth buffer precision dramatically. A good default is `10`.
:::

## Related Pages
- [Scene System](../../index.md)
- [Post Processing](../../../systems/post-processing/index.md)
- [UI Screen Panel](../../../systems/ui/screenpanel.md)