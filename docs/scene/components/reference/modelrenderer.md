---
title: "Model Renderer"
sources:
  - engine/Sandbox.Engine/Scene/Components/Render/ModelRenderer.cs
updated: 2026-04-25
created: 2026-04-27
---

# Model Renderer

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `ModelRenderer` component.

The `ModelRenderer` component gives a GameObject a visual representation by rendering a 3D model in the world.

## Quick Working Example

```csharp
public class SpawnBox : Component
{
	protected override void OnStart()
	{
		// Add a model renderer to this game object
		var modelRenderer = Components.GetOrCreate<ModelRenderer>();
		
		// Set the model to a built-in dev asset
		modelRenderer.Model = Model.Load( "models/dev/box.vmdl" );
		
		// Change the color of the model
		modelRenderer.Tint = Color.Red;
	}
}
```

### Changing the Model at Runtime

You can easily swap the model at runtime using the `Model` property.

```csharp
var renderer = Components.Get<ModelRenderer>();
if ( renderer != null )
{
	renderer.Model = Model.Load( "models/dev/sphere.vmdl" );
}
```

### Changing the Color

Because `ModelRenderer` implements the `ITintable` interface, you can change its color uniformly.

```csharp
var renderer = Components.Get<ModelRenderer>();
renderer.Tint = new Color( 0.5f, 0.8f, 1.0f ); // Light blue
```

### Body Groups

Some models are broken up into parts called "Body Groups" (like a car with different bumper options). You can enable or disable these parts.

```csharp
var renderer = Components.Get<ModelRenderer>();

// Turn on a specific body group by name
renderer.SetBodyGroup( "bumper", 1 );

// Disable it
renderer.SetBodyGroup( "bumper", 0 );
```

## Configuration

| Property | Description |
|:---|:---|
| `Model` | The `Model` resource to render. |
| `Tint` | The color tint applied to the model. |
| `MaterialOverride` | Replaces the model's default material with a custom one. |
| `RenderType` | Determines how the model casts shadows (e.g., On, Off, ShadowsOnly). |
| `BodyGroups` | Bitmask defining which optional parts of the model are visible. |

## Troubleshooting

:::danger "My model is invisible!"
1. Check that the `Model` property is actually set and not null.
2. Verify the `GameObject` is enabled in the scene hierarchy.
3. If setting the model via code, ensure the path to the model is correct (e.g., `"models/dev/box.vmdl"`).
:::

## Related Pages
- [Component Basics](../index.md)
- [Component Interfaces](../component-interfaces/index.md)
