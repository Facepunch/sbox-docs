---
title: "Text Renderer"
icon: "🔤"
sources:
  - engine/Sandbox.Engine/Scene/Components/Render/TextRenderer.cs
updated: 2026-04-25
created: 2026-04-27
---

# Text Renderer

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `TextRenderer` component.

The `TextRenderer` component displays 3D text in the world, perfect for floating damage numbers, name tags, or diegetic UI.

## Quick Working Example

```csharp
using Sandbox;
using System;

public sealed class FloatingDamageNumber : Component
{
	[RequireComponent] public TextRenderer Text { get; set; }

	public void ShowDamage( int amount, Color color )
	{
		Text.Text = $"-{amount}";
		Text.Color = color;
		Text.FontSize = 64f;
		Text.FontFamily = "Poppins";
		Text.FontWeight = 800; // Bold
	}
	
	protected override void OnUpdate()
	{
		// Float upwards and fade out
		GameObject.WorldPosition += Vector3.Up * 50f * Time.Delta;
		Text.Color = Text.Color.WithAlpha( Text.Color.a - Time.Delta );
		
		if ( Text.Color.a <= 0 )
			GameObject.Destroy();
	}
}
```

### Changing Text Alignment

By default, text is centered on the GameObject's origin. You can adjust the horizontal and vertical alignment if you want the text to expand in a specific direction.

```csharp
var textRenderer = Components.GetOrCreate<TextRenderer>();

textRenderer.Text = "Left Aligned";
textRenderer.HorizontalAlignment = TextRenderer.HAlignment.Left;
textRenderer.VerticalAlignment = TextRenderer.VAlignment.Bottom;
```

## Configuration

| Property | Description |
|:---|:---|
| `Text` | The string of text to render. |
| `Color` | The color of the text. |
| `FontSize` | The internal resolution/size of the font. *See troubleshooting below.* |
| `FontFamily` | The name of the font to use (e.g., "Poppins", "Roboto"). |
| `FontWeight` | The boldness of the font (e.g., 400 for Normal, 800 for Bold). |
| `Scale` | The physical size of the text in the 3D world. |
| `HorizontalAlignment` | How the text is aligned horizontally relative to the GameObject's origin (Left, Center, Right). |
| `VerticalAlignment` | How the text is aligned vertically relative to the GameObject's origin (Top, Center, Bottom). |
| `BlendMode` | How the text blends with the background. Default is `Normal`. |
| `FogStrength` | How much the text is affected by atmospheric fog. `1` is fully affected, `0` ignores fog. |

## Troubleshooting

:::danger My text is blurry!
A common mistake is trying to make text bigger in the world by increasing the `FontSize` property. `FontSize` acts more like the "texture resolution" of the text. 

If you want the text to take up more physical space in the 3D world, increase the **`Scale`** property (or the GameObject's transform scale) while keeping the `FontSize` at a reasonable number (like 32 or 64). If you set `FontSize` to 500, the engine will try to generate a massive high-resolution texture, which can be blurry or kill performance.
:::

## Related Pages
- [Model Renderer](modelrenderer.md)
- [World Panel](../../../systems/ui/worldpanel.md)
