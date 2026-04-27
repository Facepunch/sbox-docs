---
title: "Sprite Renderer"
icon: "❤️"
sources:
  - engine/Sandbox.Engine/Scene/Components/Render/SpriteRenderer.cs
updated: 2026-04-25
created: 2026-04-27
---

# Sprite Renderer

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `SpriteRenderer` component.

The `SpriteRenderer` component allows you to render 2D sprites in your 3D world. It supports playing flipbook animations, responding to animation events, and standard billboard behaviors to always face the camera.

## Quick Working Example

```csharp
public class FloatingCoin : Component
{
	[Property] public SpriteRenderer Renderer { get; set; }
	
	public void Collect()
	{
		// Play a one-shot "Collect" animation
		Renderer.PlayAnimation( "Collect" );
		
		// When the animation finishes, destroy the object
		Renderer.OnAnimationEnd = ( animName ) =>
		{
			if ( animName == "Collect" )
			{
				GameObject.Destroy();
			}
		};
	}
}
```

### Responding to Animation Events

When playing a sprite animation, you might want to execute logic on specific frames or when the animation ends.

```csharp
var renderer = Components.Get<SpriteRenderer>();

// Fired when the animation finishes playing (or loops)
renderer.OnAnimationEnd = ( animName ) =>
{
	Log.Info( $"Animation {animName} just finished!" );
};

// Fired on specific frames if you added 'Broadcast Messages' to the sprite asset
renderer.OnBroadcastMessage = ( message ) =>
{
	if ( message == "footstep" )
	{
		Sound.Play( "sounds/footstep.sound" );
	}
};
```

### Pixel Art Settings

If you are rendering low-resolution pixel art, the default bilinear filtering will make it look blurry. 
Ensure you set `TextureFilter` to `Point` so the pixels remain sharp and crisp.

## Configuration

| Property | Description |
|---|---|
| `Sprite` | The `Sprite` resource asset to render. |
| `StartingAnimationName` | The animation that plays automatically when the scene starts. |
| `PlaybackSpeed` | The playback speed multiplier. `0` pauses it, negative values play in reverse. |
| `Size` | The width and height of the sprite in world units. |
| `Color` | A tint color multiplied with the sprite's texture. |
| `Additive` | If true, the sprite is rendered additively (good for glowing effects). |
| `Shadows` | Whether or not the sprite should cast 3D shadows. |
| `Opaque` | If true, the sprite is fully opaque. Semi-transparent pixels are dithered. |
| `AlphaCutoff` | When `Opaque` is true, pixels with an alpha below this threshold are discarded. |
| `Lighting` | If true, the sprite receives lighting from the scene. Otherwise, it is fullbright. |
| `TextureFilter` | How the texture is sampled. Use `Point` for crisp pixel art, or `Bilinear` for smooth images. |
| `Billboard` | How the sprite faces the camera (`Always`, `YOnly`, `Particle`, or `None`). |
| `FogStrength` | How much the sprite blends with volumetric fog in the scene. |

## Troubleshooting

:::warning Blurry Pixel Art
If your 2D game looks soft or blurry, find your `SpriteRenderer` and change the **Texture Filter** from `Bilinear` to `Point`. 
:::
