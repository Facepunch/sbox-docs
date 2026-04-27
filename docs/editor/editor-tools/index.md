---
title: "Editor Tools"
icon: "🪛"
created: 2023-12-29
updated: 2026-04-25
---

# Editor Tools

You can create your own editor tool to help you create your game by writing standard C# classes. Your tool needs to be created in an [editor project](../editor-project.md).

## Quick Working Example

Here is a basic tool that logs when it is enabled and disabled.

```csharp
[EditorTool] // this class is an editor tool
[Title( "Rocket" )] // title of your tool
[Icon( "rocket_launch" )] // icon name from https://fonts.google.com/icons?selected=Material+Icons
[Shortcut( "editortool.rocket", "u" )] // keyboard shortcut
public class MyRocketTool : EditorTool
{
	public override void OnEnabled()
	{

	}

	public override void OnDisabled()
	{

	}

	public override void OnUpdate()
	{
		// Draw a line connecting the origin to where the user is looking
		Gizmo.Draw.Line( Vector3.Zero, Gizmo.CurrentRay.Position + Gizmo.CurrentRay.Forward * 1000f );
	}
}
```

This will create a tool that you can select here.

![](./images/this-will-create-a-tool-that-you-can-select-here.png)

### 1. Interacting with the Scene

The EditorTool has a member called `Scene` to access the scene.

```csharp
public override void OnUpdate()
{
	var tr = Scene.Trace.Ray( Gizmo.CurrentRay, 5000 )
					.UseRenderMeshes( true )
					.UsePhysicsWorld( false )
					.WithoutTags( "sprinkled" )
					.Run();

	if ( tr.Hit )
	{
		using ( Gizmo.Scope( "cursor" ) )
		{
			Gizmo.Transform = new Transform( tr.HitPosition, Rotation.LookAt( tr.Normal ) );
			Gizmo.Draw.LineCircle( 0, 100 );
		}
	}
}
```

### 2. Preventing Selection

Depending on how your tool operates, you might want to prevent the user's ability to click to select [GameObjects](../../scene/gameobject.md) in the scene. To do this you can change `AllowGameObjectSelection` on your tool.

```csharp
public override void OnEnabled()
{
	AllowGameObjectSelection = false;
}
```

### 3. Creating Overlay UI

You can create UI on the scene's overlay. This is useful for creating controls and other things.

```csharp
public override void OnEnabled()
{
    // create a widget window. This is a window that  
    // can be dragged around in the scene view
	var window = new WidgetWindow( SceneOverlay );
	window.Layout = Layout.Column();
	window.Layout.Margin = 16;
 
    // Create a button for us to press
	var button = new Button( "Shoot Rocket" );
	button.Pressed = () => Log.Info( "Rocket Has Been Shot!" );

    // Add the button to the window's layout
	window.Layout.Add( button );

    // Calling this function means that when your tool is deleted,
    // ui will get properly deleted too. If you don't call this and
    // you don't delete your UI in OnDisabled, it'll hang around forever.
	AddOverlay( window, TextFlag.RightTop, 10 );
}
```

The UI is created when the tool is activated, and destroyed when it's deactivated.

![](./images/the-ui-is-created-when-the-tool-is-activated-and-destroyed.png)

## Troubleshooting

:::warning "My Editor Tool doesn't appear in the toolbar!"
Make sure your class inherits from `EditorTool`, has the `[EditorTool]` attribute, and is located within an **Editor Project**. Tools written in a standard Game Project will not be loaded into the editor's toolbars.
:::

:::danger "My Overlay UI stays on screen forever!"
If you create UI using `WidgetWindow` or similar classes, you **must** call `AddOverlay()` or manually delete/destroy the UI in your tool's `OnDisabled()` method. Otherwise, the UI will leak and persist even after the user switches to a different tool.
:::

## Related Pages
- [Editor Project](../editor-project.md)
- [Custom Editors](../custom-editors.md)
