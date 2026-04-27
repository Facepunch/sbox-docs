---
title: "HudPainter"
sources:
  - engine/Sandbox.Engine/Systems/Render/CommandList/HudPainter/HudPainter.cs
icon: "🎨"
created: 2024-11-07
updated: 2025-06-28
---

# HudPainter

> **New to UI?** Reference for the `HudPainter` immediate-mode drawing API; UI overview is at **[UI](index.md)**.

`HudPainter` is an immediate-mode 2D drawing API used to render simple shapes, lines, text, and textures directly to a camera's HUD.

## Quick Start

You can draw directly to the screen every frame from any component's `OnUpdate` method by grabbing the camera's `Hud`.

```csharp
protected override void OnUpdate()
{
	if ( Scene.Camera is null )
		return;

	var hud = Scene.Camera.Hud;

	// Draw a red circle and a white line
	hud.DrawCircle( new Vector2( 100, 100 ), new Vector2( 50, 50 ), Color.Red );
	hud.DrawLine( new Vector2( 100, 100 ), new Vector2( 200, 200 ), 5, Color.White );

	// Draw some text
	hud.DrawText( "Hello World!", 24, Color.White, new Vector2( 150, 150 ) );
}
```

### Drawing Rectangles and Borders

You can draw basic shapes like rectangles, optionally specifying border widths and corner radii.

```csharp
var hud = Scene.Camera.Hud;
var myRect = new Rect( Screen.Width / 2 - 50, Screen.Height / 2 - 50, 100, 100 );

// Draw a blue box with rounded corners and a white border
hud.DrawRect( 
	myRect, 
	Color.Blue, 
	cornerRadius: new Vector4( 8, 8, 8, 8 ), 
	borderWidth: new Vector4( 2, 2, 2, 2 ), 
	borderColor: Color.White 
);
```

### Drawing Images and Textures

You can easily render an image texture to a specific rectangle on screen.

```csharp
// Assuming you have a texture loaded
Texture myTexture = Texture.Load( FileSystem.Mounted, "textures/ui/crosshair.png" );

var hud = Scene.Camera.Hud;
var rect = new Rect( Mouse.Position, new Vector2( 32, 32 ) );

// Draw the texture, optionally tinting it (red in this case)
hud.DrawTexture( myTexture, rect, Color.Red );
```

### Advanced Text Rendering

For more complex text formatting, you can construct a `TextRendering.Scope`.

```csharp
var hud = Scene.Camera.Hud;

var scope = new TextRendering.Scope( "Player 1", Color.Green, 32 );
var rect = new Rect( 50, 50, 200, 50 );

// Draw text aligned to the center right of the defined rect
hud.DrawText( scope, rect, TextFlag.Center | TextFlag.Right );
```

## API Reference

Here is a summary of the available `HudPainter` methods:

| Method | Description |
|--------|-------------|
| `DrawCircle( position, size, color )` | Draws a filled circle at the specified position and size. |
| `DrawRect( rect, color, ... )` | Draws a rectangle with optional parameters for `cornerRadius`, `borderWidth`, and `borderColor`. |
| `DrawLine( start, end, width, color )` | Draws a line of a specific thickness between two points. |
| `DrawTexture( texture, rect, [tint] )` | Renders a `Texture` to a screen rectangle, optionally tinted. |
| `DrawText( text, size, color, pos/rect )`| Draws simple text at a point or within a rectangle. |
| `SetBlendMode( mode )` | Sets the blend mode for subsequent drawing operations. |
| `SetMatrix( matrix )` | Sets the transformation matrix for subsequent drawing operations. |

## Troubleshooting

**The text or shapes are rendering behind my UI panels!**  
`Scene.Camera.Hud` renders before the main Panel system UI. If you want to render *over* the standard UI, use `Scene.Camera.Overlay` which functions identically but draws after panels.

**`NullReferenceException` when accessing `Scene.Camera.Hud`**  
Ensure your Scene actually has an active Camera component. If you are running code before the camera is initialized, `Scene.Camera` will be null. Always check `if ( Scene.Camera is null ) return;` first.

**Nothing is showing up on the screen**  
Make sure you are calling these draw functions every frame, typically inside `OnUpdate()`. `HudPainter` does not retain drawn objects; they exist only for the frame they are drawn.

## Related Pages

- [Razor Panels](razor-panels/index.md) - For fully featured UI with HTML/CSS.
- [VirtualGrid](virtualgrid.md) - For displaying many repeating UI items.
