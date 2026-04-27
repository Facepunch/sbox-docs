---
title: "Glyphs"
icon: "🎨"
sources:
  - engine/Sandbox.Engine/Systems/Input/Input.Glyphs.cs
created: 2026-04-27
updated: 2026-04-27
---

# Glyphs

Input glyphs automatically return the correct button texture for an action, seamlessly adapting to whatever controller or keyboard the player is currently using.

## Quick Working Example

In Razor UI panels, you can directly use `Input.GetGlyph()` as the source for an `<Image>` element.

```html
<!-- Display the current button for the "jump" action -->
<Image Texture="@Input.GetGlyph( "jump", InputGlyphSize.Medium, GlyphStyle.Light )" />
```

### Retrieving an Action Glyph
The most common use case is fetching the texture mapped to an action string from your project settings.

```csharp
// Returns the default small glyph texture for the "attack" action
Texture attackTexture = Input.GetGlyph( "attack" );
```

### Retrieving an Analog Glyph
You can also request a glyph representing a specific physical analog input, such as a controller's thumbstick or trigger.

```csharp
// Returns a texture representing the Left Joystick
Texture moveTexture = Input.GetGlyph( InputAnalog.LeftStickX );
```

### Checking Specific Key Glyphs
If you need to show a glyph for a specific, hardcoded keyboard key rather than an action, use the Keyboard API.

```csharp
// Returns a texture of the physical 'W' key
Texture wKeyTexture = Input.Keyboard.GetGlyph( "W" );
```

## Configuration

The `GetGlyph` method takes optional arguments to control the visual style and resolution of the returned texture.

### Sizes

You can request glyphs in different native resolutions using the `InputGlyphSize` enum.

| Size | Resolution | Usage |
|------|------------|-------|
| `InputGlyphSize.Small` | 32x32 | Default. Good for inline text or small UI hints. |
| `InputGlyphSize.Medium` | 128x128 | Good for standard menus and prompts. |
| `InputGlyphSize.Large` | 256x256 | High-resolution, good for large on-screen tutorials. |

### Styles

You can customize the look of the glyphs to match your game's UI theme using the `GlyphStyle` struct.

| Style | Description |
|-------|-------------|
| `GlyphStyle.Knockout` | Default. Colored labels/outlines on a knocked-out background. |
| `GlyphStyle.Light` | Black details/borders on a solid white background. |
| `GlyphStyle.Dark` | White details/borders on a solid black background. |
| `.WithNeutralColorABXY()` | Removes the specific face button colors (e.g., green A, red B), replacing them with the base style color. |
| `.WithSolidABXY()` | Face buttons receive a solid color fill. |

Example:
```csharp
var style = GlyphStyle.Dark.WithSolidABXY();
Texture darkSolidJump = Input.GetGlyph( "jump", InputGlyphSize.Medium, style );
```

*(Note: There is also an older legacy overload `Input.GetGlyph("action", outline: true)` which functions similarly to knockout.)*

## Troubleshooting

:::tip Do Not Cache Glyphs
Because players can unplug a controller or switch to a keyboard at any moment, the correct glyph might change instantly. Do not cache the resulting `Texture` in an `OnStart` method. Always call `Input.GetGlyph()` dynamically (e.g., in `OnUpdate` or inside your Razor view). It is a very cheap operation to call every frame.
:::

:::warning Unbound Actions
If a player has unbound an action, or if you request a glyph for an action name that doesn't exist, `Input.GetGlyph()` will return a fallback "UNBOUND" texture. If you see this in your UI, double-check your action spelling.
:::

## Related Pages
- [Input (Actions)](index.md)
- [Raw Input](raw-input.md)
- [Controller Input](controller-input.md)
- [UI Panels](../ui/index.md)