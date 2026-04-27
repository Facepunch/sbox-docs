---
title: "Editor Apps"
icon: "📱"
created: 2023-12-29
updated: 2026-04-15
sources:
  - engine/Sandbox.Tools/Editor/EditorAppAttribute.cs
  - engine/Sandbox.Tools/Qt/Window.cs
---

# Editor Apps

Editor Apps are standalone tools that run entirely within the engine's Editor, providing dedicated windows for tasks like editing assets or monitoring systems.

## Quick Working Example

```csharp
using Sandbox;
using Editor;

// 1. Add the [EditorApp] attribute to make it discoverable in the Editor
[EditorApp( "My Tool", "build", "A custom tool for my game" )]
public class MyCustomEditorApp : Window
{
	public MyCustomEditorApp()
	{
		// 2. Set basic window properties
		WindowTitle = "My Tool";
		SetWindowIcon( "build" );
		MinimumSize = new Vector2( 400, 300 );
		
		// 3. Set up the UI layout using a Canvas
		Canvas = new Widget( null );
		Canvas.Layout = Layout.Column();
		Canvas.Layout.Margin = 16;
		Canvas.Layout.Spacing = 8;
		
		// 4. Add interactive widgets
		Canvas.Layout.Add( new Label( "Welcome to my custom tool!" ) );
		
		var button = Canvas.Layout.Add( new Button( "Click Me" ) );
		button.Clicked += () =>
		{
			Log.Info( "Button was clicked in the custom tool!" );
		};
		
		Canvas.Layout.AddStretchCell();
	}
}
```

### Launching

Once you compile a class with the `[EditorApp]` attribute, it automatically becomes available in the Editor. You can launch it by navigating to **View -> Apps** in the top menu bar, or by finding it in the App sidebar.

![The app will be available on the App sidebar and the Apps menu.](./images/the-app-will-be-available-on-the-app-sidebar-and-the-apps-me.png)

### Building the UI

Because Editor Apps inherit from `Window`, you cannot modify their `Layout` directly. Instead, you create a root `Widget` and assign it to the window's `Canvas` property, then add child widgets (like `Label`, `Button`, or `TextEntry`) to the canvas's layout.

```csharp
Canvas = new Widget( null );
Canvas.Layout = Layout.Column();

Canvas.Layout.Add( new Label( "First Name:" ) );
var nameInput = Canvas.Layout.Add( new LineEdit() );
```

## Configuration

The `[EditorApp]` attribute requires a few basic parameters to integrate your app into the Editor's menus.

| Parameter | Type | Description |
|---|---|---|
| `Title` | `string` | The display name of your app in the menus. |
| `Icon` | `string` | The Material Design icon identifier (e.g., `"pregnant_woman"`, `"build"`). |
| `Description` | `string` | A short description explaining what the app does, shown in tooltips. |

## Troubleshooting

:::danger "My Window crashes immediately!"
If you attempt to modify the `Layout` property of the `Window` itself, it will fail. You must assign a new `Widget` to the `Canvas` property and modify the `Canvas.Layout`.
:::

:::warning "My App isn't showing up in the menu"
Make sure your class inherits from `Editor.Window` and is public. It also helps to ensure there are no compilation errors in your project, as the Editor will not reload code with compile errors.
:::

## Related Pages
- [Editor Widgets](editor-widgets.md)
- [Custom Editors](custom-editors.md)
