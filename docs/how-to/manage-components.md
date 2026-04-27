---
title: "Manage Components"
icon: "⚙️"
sources:
  - engine/Sandbox.Engine/Scene/GameObject/ComponentList.cs
  - engine/Sandbox.Engine/Scene/Components/Component.cs
created: 2026-04-27
updated: 2026-04-27
---

# Manage Components

To dynamically attach, configure, or remove components from GameObjects at runtime, use the `GameObject.Components` API.

## Quick Working Example

```csharp
using Sandbox;

public sealed class ComponentManager : Component
{
	protected override void OnStart()
	{
		// Add a new component
		var modelRenderer = GameObject.Components.Create<ModelRenderer>();
		modelRenderer.Model = Model.Load( "models/dev/box.vmdl" );

		// Get an existing component, or create it if missing
		var rigidbody = GameObject.Components.GetOrCreate<Rigidbody>();
		
		// Get a component (returns null if not found)
		var existingRenderer = GameObject.Components.Get<ModelRenderer>();
		if ( existingRenderer != null )
		{
			// Modify component properties
			existingRenderer.Tint = Color.Red;
		}

		// Remove/Destroy the component
		if ( rigidbody != null )
		{
			rigidbody.Destroy();
		}
	}
}
```

### Adding Components

To attach a new component to a GameObject, use the `Create<T>()` method.

```csharp
// Creates a new PointLight component on the GameObject
var light = GameObject.Components.Create<PointLight>();
light.LightColor = Color.Blue;
light.Radius = 400f;
```

If you only want to create the component if it doesn't already exist, use `GetOrCreate<T>()`. This is very useful when you want to ensure a GameObject has a specific capability without adding duplicates.

```csharp
// Will find an existing Rigidbody, or create a new one if it's missing
var rigidbody = GameObject.Components.GetOrCreate<Rigidbody>();
```

### Removing Components

To remove a component, you do not remove it from the `Components` list directly. Instead, you call the `Destroy()` method on the component itself. The engine will safely clean it up and remove it from the list.

```csharp
var renderer = GameObject.Components.Get<ModelRenderer>();

if ( renderer != null )
{
	// Tell the component to destroy itself
	renderer.Destroy();
}
```

### Toggling Components

Instead of destroying and recreating components, it's often more efficient to enable or disable them. A disabled component will not run its `OnUpdate` loop and built-in components (like renderers or colliders) will stop functioning.

```csharp
var light = GameObject.Components.Get<PointLight>();

if ( light != null )
{
	// Turn off the light
	light.Enabled = false;
	
	// Turn the light back on
	light.Enabled = true;
}
```

The `GameObject.Components` list (`ComponentList`) provides the following common methods to manage components on a specific GameObject. For a full list of queries to find components across the hierarchy, see [Component Queries](component-queries.md).

| Method | Description |
|---|---|
| `Create<T>()` | Creates and attaches a new component of type `T` to the GameObject. |
| `GetOrCreate<T>()` | Returns the first component of type `T` if it exists; otherwise, creates and attaches a new one. |
| `Get<T>()` | Returns the first component of type `T` if it exists; otherwise, returns `null`. |
| `TryGet<T>( out T component )` | Returns `true` and sets the `out` parameter if the component exists; otherwise returns `false`. |
| `GetAll<T>()` | Returns an `IEnumerable<T>` of all components of type `T` on the GameObject. |
| `Remove( Component component )` | Removes the specific component instance from the GameObject. Usually, it's preferred to just call `component.Destroy()`. |

## Troubleshooting

:::danger Destroying the GameObject by Mistake
Make sure you call `Destroy()` on the component instance (e.g. `myComponent.Destroy()`), not `GameObject.Destroy()`. Calling `Destroy()` on the GameObject will destroy the entire object and all of its components.
:::

:::warning Components list is not a standard List
You cannot use standard C# `List.Add()` or `List.Remove()` methods on `GameObject.Components`. You must use `Create<T>()` to add and `Destroy()` on the component instance to remove.
:::

## Related Pages
- [Component Queries](component-queries.md)
- [Create a Component](create-a-component.md)
- [GameObject](../scene/gameobject.md)
