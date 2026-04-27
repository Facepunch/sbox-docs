---
title: "Load Image from URL"
icon: "🖼️"
sources:
  - engine/Sandbox.Engine/Resources/Textures/Texture.Load.cs
  - engine/Sandbox.Engine/Systems/UI/Controls/Image.cs
created: 2026-04-27
updated: 2026-04-27
---

# Load Image from URL

How to download and display an image from the internet at runtime.

## Quick Working Example

If you are building a user interface in Razor, the `<Image>` panel can load images directly from an HTTP URL using the `src` attribute.

```html
@using Sandbox;
@using Sandbox.UI;
@inherits PanelComponent

<root>
	<!-- This will automatically download the image asynchronously and display it -->
	<Image src="https://sbox.game/img/sbox-logo-square.svg" class="logo" />
</root>

<style>
	.logo {
		width: 128px;
		height: 128px;
		background-color: transparent;
	}
</style>
```

### Loading a Texture in C#

If you need to fetch a texture from the web programmatically (for a 3D model, material, or custom UI), use `Texture.LoadAsync()`. This method fetches the image on a background thread and returns the texture when it is fully loaded.

```csharp
using Sandbox;
using System.Threading.Tasks;

public sealed class DynamicBillboard : Component
{
	[Property] public ModelRenderer Renderer { get; set; }

	protected override void OnStart()
	{
		// Start the download immediately without freezing the game
		_ = DownloadAndApplyTexture();
	}

	async Task DownloadAndApplyTexture()
	{
		// Await the download to finish
		Texture downloadedTexture = await Texture.LoadAsync( "https://sbox.game/img/sbox-logo-square.svg" );

		if ( downloadedTexture != null && Renderer != null )
		{
			// Apply the loaded texture to the model's material
			Renderer.Set( "Color", downloadedTexture );
		}
	}
}
```

### Displaying Steam Avatars

You can load a player's Steam avatar if you know their 64-bit SteamID using `Texture.LoadAvatar`.

```csharp
// Size can be 32, 64, or 128
long steamId = 76561197960287930;
Texture avatar = Texture.LoadAvatar( steamId, 64 );
```

In a Razor panel, you can assign this dynamically:

```html
@using Sandbox;
@using Sandbox.UI;
@inherits PanelComponent

<root>
	<Image Texture="@AvatarTexture" />
</root>

@code
{
	public Texture AvatarTexture { get; set; }

	protected override void OnStart()
	{
		AvatarTexture = Texture.LoadAvatar( 76561197960287930, 64 );
	}
}
```

## Configuration

When using `<Image src="...">`, the underlying UI system calls `Texture.LoadAsync()` automatically. 

| Method | Description |
|---|---|
| `Texture.LoadAsync( url )` | Returns a `Task<Texture>`. Use this when loading over HTTP. |
| `Texture.Load( url )` | Blocking call. Only use this if you are absolutely certain the operation must finish before the frame continues. **Not recommended for web URLs.** |
| `Texture.LoadAvatar( steamid, size )` | Quickly fetches a cached Steam avatar. |

## Troubleshooting

:::warning "Image is not showing up!"
Ensure the URL is a direct link to an image file (e.g., ends in `.png`, `.jpg`, `.svg`). If the URL points to an HTML webpage instead of the raw image data, it will fail to load.
:::

:::danger "The game freezes when the image loads!"
If you use `Texture.Load("http...")` instead of `Texture.LoadAsync("http...")`, the entire game engine will halt and wait for the HTTP request to finish. Always use `LoadAsync` when fetching data over the network.
:::

## Related Pages
- [UI System](../systems/ui/index.md)
- [Razor Panels](../systems/ui/razor-panels/index.md)
- [Use Async Tasks](component-async.md)