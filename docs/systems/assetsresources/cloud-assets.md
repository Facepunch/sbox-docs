---
title: "Cloud Assets"
icon: "☁️"
sources:
  - engine/Sandbox.Engine/Services/Packages/Cloud/Cloud.cs
created: 2026-04-27
updated: 2026-04-27
---

# Cloud Assets

There is a large selection of Assets (Textures, Models, Sounds, etc.) available on sbox.game that you can reference directly in your game without manually downloading or mounting files.

## Quick Working Example

You can load a cloud asset asynchronously at runtime using `Cloud.Load<T>()`.

```csharp
using Sandbox;
using System.Threading.Tasks;

public class CloudSpawner : Component
{
	protected override async Task OnLoad()
	{
		// Download and load a model from the cloud at runtime
		var model = await Cloud.Load<Model>( "facepunch.w_usp" );
		
		if ( model != null )
		{
			// Apply the loaded model to a ModelRenderer
			var renderer = Components.GetOrCreate<ModelRenderer>();
			renderer.Model = model;
		}
	}
}
```

### Referencing in Editor Properties

If you expose your variables as Properties on a Component, you can use the Cloud Browser in the Editor to assign them.

```csharp
// Can reference a Local Model OR a Cloud Model via the Inspector
[Property] public Model WeaponModel { get; set; } 
```

### Referencing via Ident for Compile-Time Packaging

If you know the ident/url of a cloud asset, you can use the `Cloud.*` functions. This downloads the asset during code compilation, embedding it as part of your shipped package.

```csharp
var myModel = Cloud.Model( "facepunch.w_usp" ); // Dot notation
var myMat = Cloud.Material( "facepunch/w_usp" ); // Slash notation
```

### Loading at Runtime

If you need to load content dynamically while the game is running (like procedural generation or a sandbox game mode), use `Cloud.Load<T>()`.

```csharp
// Load a sound event from the cloud
var sound = await Cloud.Load<SoundEvent>( "facepunch.fart" );
```

## Configuration

The `Cloud` static class provides several helper methods for common types.

| Method | Returns | Description |
|:---|:---|:---|
| `Cloud.Model(ident)` | `Model` | Loads a cloud model during code compilation. |
| `Cloud.Material(ident)` | `Material` | Loads a cloud material during code compilation. |
| `Cloud.SoundEvent(ident)` | `SoundEvent` | Loads a cloud sound event during code compilation. |
| `Cloud.Shader(ident)` | `Shader` | Loads a cloud shader during code compilation. |
| `Cloud.Load<T>(ident, withCode)` | `Task<T>` | Asynchronously downloads and mounts a cloud asset at runtime. |

## Troubleshooting

:::danger "My runtime model is null!"
If `Cloud.Load<T>()` returns null, ensure you are passing the correct package ident (e.g., `"facepunch.w_usp"`) and that the package exists on sbox.game and is public.
:::

:::warning "Asset changes aren't updating!"
If you used `Cloud.Model` or assigned it via the Inspector, the asset was downloaded locally during compile time. If the author updates the asset on sbox.game, it will not automatically update in your shipped game until you re-compile and re-publish your project.
:::

## Related Pages
* [Assets/Resources Overview](index.md)
* [Custom Assets](custom-assets.md)

