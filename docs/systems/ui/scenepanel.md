---
title: "Scene Panel"
icon: "📹"
sources:
  - engine/Sandbox.Engine/Systems/UI/Controls/ScenePanel.cs
updated: 2026-04-25
created: 2026-04-27
---

# Scene Panel

> **New to UI?** Reference for the `ScenePanel` component; UI overview is at **[UI](index.md)**.

`ScenePanel` allows you to render a separate 3D `Scene` directly onto a 2D UI panel, useful for things like character loadout screens, minimaps, or 3D item viewers.

## Quick Working Example

```csharp
@using Sandbox;
@using Sandbox.UI;
@inherits PanelComponent

<root>
	<div class="viewer-container">
		<ScenePanel RenderScene=@MyScene />
	</div>
</root>

@code
{
	// The 3D scene we want to render inside the UI
	public Scene MyScene { get; set; }

	protected override void OnStart()
	{
		// 1. Create a new headless Scene
		MyScene = new Scene() { WantsSystemScene = false };

		// 2. Load a scene file into it
		var options = new SceneLoadOptions { ShowLoadingScreen = false };
		if ( options.SetScene( "scenes/item_viewer.scene" ) )
		{
			MyScene.Load( options );
		}
	}
}
```

### Rendering a Scene from Code

You don't have to use Razor or existing `.scene` files to use a `ScenePanel`. You can create a `ScenePanel`, give it a new `Scene`, and populate that scene entirely via code.

```csharp
public sealed class CharacterViewer : PanelComponent
{
	ScenePanel scenePanel;

	protected override void OnTreeFirstBuilt()
	{
		scenePanel = new ScenePanel();
		scenePanel.Parent = Panel;

		// The ScenePanel creates a default empty RenderScene for us
		var scene = scenePanel.RenderScene;

		// Add a camera to the scene so it has a viewpoint
		var camObj = new GameObject( true, "Camera" );
		camObj.Parent = scene;
		camObj.WorldPosition = new Vector3( 100, 0, 50 );
		camObj.WorldRotation = Rotation.From( 0, 180, 0 );
		var cam = camObj.Components.Create<CameraComponent>();

		// Add a model to look at
		var modelObj = new GameObject( true, "Model" );
		modelObj.Parent = scene;
		var renderer = modelObj.Components.Create<ModelRenderer>();
		renderer.Model = Model.Load( "models/citizen/citizen.vmdl" );
	}
}
```

### Rendering Once (Performance Optimization)

If you are rendering a static 3D object (like an item icon) that never animates or changes, rendering it every single frame is a waste of performance. You can use the `RenderOnce` property to draw it once and cache the result.

```csharp
// The scene will render once and then freeze the image
scenePanel.RenderOnce = true;

// If you change the item later, you can force it to update the image once more
scenePanel.RenderNextFrame();
```

## Configuration

| Property | Description |
|---|---|
| `RenderScene` | The `Scene` object that this panel is responsible for simulating and rendering. |
| `RenderOnce` | If true, the scene will only be rendered once and the resulting texture will be cached. |
| `RenderTexture` | (Read-only) The actual generated `Texture` object the panel is drawing. |

## Troubleshooting

:::danger "My ScenePanel is completely black/empty!"
Ensure that the `Scene` you are passing into `RenderScene` actually has a `CameraComponent` in it, and that there are lights illuminating your objects. Without a camera, the panel doesn't know from where to render the scene.
:::

:::warning "The ScenePanel is rendering, but nothing is moving!"
If you have `RenderOnce` set to `true`, the scene will only render the very first frame. If you need animations or particle effects to play, ensure `RenderOnce` is `false`.
:::

## Related Pages
- [Scene System](../../scene/index.md)
- [Razor Panels](../ui/razor-panels/index.md)
