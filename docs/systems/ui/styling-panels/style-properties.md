---
title: "Style Properties"
icon: "🎨"
created: 2024-09-24
updated: 2026-03-23
---

# Style Properties

We try to keep as close to standard web styles as possible - but not every property is implemented. We'll use this page to highlight any differences.

Anything marked in **bold\*** may behave differently to how you'd expect on web.

## Common Types

| Name | Description | Examples |
|------|-------------|----------|
| Float | A standard float | `flex-grow: 2.5;` |
| String | A string with or without quotes | `font-family: Poppins;`,`content: "Back to Menu";` |
| int  | A standard int | `font-weight: 600;` |
| Color | Can have alpha | `color: #fff;`,`color: #ffffffaa;`,`color: rgba(red, 0.5);` |
| Length | A dimension, pixel or relative. | `left: 10px;`,`left: 10%;`,`left: 10em;`,`left: 10vw;`,`mask-angle: 10deg;` |

## Custom Style Properties

| Name | Parameters | Examples / Notes |
|------|------------|------------------|
| aspect-ratio | Float, Float (Optional) | `aspect-ratio: 1;`,`aspect-ratio: 16/9;` |
| background-image-tint | Color      | Multiplies the `background-image` by this Color. Not a replacement for `filter` or `backdrop-filter`. |
| border-image-tint | Color      | Multiplies the `border-image` by this Color. |
| mask-scope | default / filter | `default` will apply the mask normally, `filter` will use the mask to blend between unfiltered and filtered. |
| sound-in | String     | The name of the sound to play when this style is applied to an element. This is useful to put on a `:hover` or `:active` style to play a sound on hover/click |
| sound-out | String     | The name of a sound to play when this style is removed from an element. |
| text-stroke | Length, Color | This will put an outline |
| text-stroke-color | Color      |                  |
| text-stroke-width | Length     |                  |

## Supported Style Properties

| Name | Parameters | Examples / Notes |
|------|------------|------------------|
| align-content | autoflex-endflex-startcenterstretchspace-betweenspace-aroundbaseline |                  |
| align-items | Same as `align-content` |                  |
| align-self | Same as `align-content` |                  |
| animation | Fills in the properties below |                  |
| animation-delay | Float      |                  |
| animation-direction | normal (default) reverse alternate alternate-reverse |                  |
| animation-duration | Float      |                  |
| animation-fill-mode | none forwardsbackwardsboth |                  |
| animation-iteration-count | int / infinite |                  |
| animation-name | String     |                  |
| animation-play-state | runningpaused |                  |
| animation-timing-function | linear (default) ease ease-in ease-out ease-in-out |                  |
| backdrop-filter | blur(Length) <br>saturate(Length) <br>contrast(Length) <br>brightness(Length) <br>grayscale(Length) <br>sepia(Length) <br>hue-rotate(Length) <br>invert(Length) <br>border-wrap(Length, Color) | `backdrop-filter: blur(10px) saturate(80%);` |
| backdrop-filter-blur | Length     |                  |
| backdrop-filter-brightness | Length     |                  |
| backdrop-filter-contrast | Length     |                  |
| backdrop-filter-hue-rotate | Length     |                  |
| backdrop-filter-invert | Length     |                  |
| backdrop-filter-saturate | Length     |                  |
| backdrop-filter-sepia | Length     |                  |
| background | Fills in the properties below |                  |
| background-angle | Length     |                  |
| background-blend-mode | normal lighten multiply |                  |
| background-color | Color      |                  |
| background-image | url(string) <br>linear-gradient(Color, Color) <br>radial-gradient(Color, Color) <br>conic-gradient(Color, Color) |                  |
| background-position | Length, Length (optional) | `background-position: 10px``background-position: 10px 15px` |
| background-position-x | Length     |                  |
| background-position-y | Length     |                  |
| background-repeat | no-repeat repeat-x repeat-y repeat |                  |
| background-size | Length, Length (optional) | `background-size: 10px` `background-size: 10px 15px` |
| background-size-x | Length     |                  |
| background-size-y | Length     |                  |
| border | border-width, border-style, border-color | `border: 10px solid black;` |
| border-bottom | Same as `border` |                  |
| border-bottom-color | Color      |                  |
| border-bottom-left-radius | Length     |                  |
| border-bottom-right-radius | Length     |                  |
| border-bottom-width | Length     |                  |
| border-color | Color      |                  |
| border-image | Same as `background-image` |                  |
| border-image-tint | Color      |                  |
| border-image-width-bottom | Length     |                  |
| border-image-width-left | Length     |                  |
| border-image-width-right | Length     |                  |
| border-image-width-top | Length     |                  |
| border-left | Same as `border` |                  |
| border-left-color | Color      |                  |
| border-left-width | Length     |                  |
| border-radius | Length     | `border-radius: 8px;`<br>`border-radius: 8px 0px 8px 8px;` |
| border-right | Same as `border` |                  |
| border-right-color | Color      |                  |
| border-right-width | Length     |                  |
| border-top | Same as `border` |                  |
| border-top-color | Color      |                  |
| border-top-left-radius | Length     |                  |
| border-top-right-radius | Length     |                  |
| border-top-width | Length     |                  |
| border-width | Length     |                  |
| bottom | Length     |                  |
| box-shadow | Length,<br>Length (optional),<br>Length (blur, optional),<br>Length (spread, optional),<br>Color | `box-shadow: 2px 2px 4px black;` |
| color | Color /<br>linear-gradient(Color, Color) /<br>radial-gradient(Color, Color) /<br>conic-gradient(Color, Color) |                  |
| column-gap | Length     |                  |
| content | string     | Sets the text of a Label.<br>eg. `content: "Loading…";` |
| cursor | none / pointer / progress / wait / crosshair / text / move / not-allowed / any custom cursors |                  |
| **display\*** | flex (default) / none | Everything is flex by default |
| filter | Same as `backdrop-filter` |                  |
| filter-blur | Length     |                  |
| filter-border-color | Color      |                  |
| filter-border-width | Length     |                  |
| filter-brightness | Length     |                  |
| filter-contrast | Length     |                  |
| filter-drop-shadow | Same as `box-shadow` |                  |
| filter-hue-rotate | Length     |                  |
| filter-invert | Length     |                  |
| filter-saturate | Length     |                  |
| filter-sepia | Length     |                  |
| filter-tint | Length     |                  |
| flex-basis | Length     |                  |
| flex-direction | row (default) / row-reverse / column / column-reverse |                  |
| flex-grow | Float      |                  |
| flex-shrink | Float      |                  |
| flex-wrap | nowrap (Default) / wrap / wrap-reverse |                  |
| font-color | Color      |                  |
| **font-family\*** | String     | Specify a single font, based on the name of the font itself, not the filename.<br>eg. `font-family: Comic Sans MS;` |
| font-size | Length     |                  |
| font-smooth | auto / always / never | `never` is good for pixel fonts |
| font-style | normal  (default) / italic |                  |
| font-variant-numeric | normal / tabular-nums |                  |
| font-weight | normal (default) / bold / light / bolder / lighter / black / int | `font-weight: bold;`,`font-weight: 300;` |
| gap  | Length, Length (optional) | Shorthand for `row-gap` and `column-gap`, specified the size of gutters. |
| height | Length     |                  |
| image-rendering | auto (default) / anisotropic / bilinear / trilinear / point / pixelated / nearest-neighbour |                  |
| justify-content | Same as `align-content` |                  |
| left | Length     |                  |
| letter-spacing | Length / normal |                  |
| line-height | Length     |                  |
| margin | Fills in the properties below |                  |
| margin-bottom | Length     |                  |
| margin-left | Length     |                  |
| margin-right | Length     |                  |
| margin-top | Length     |                  |
| mask | Shorthand for other mask properties |                  |
| mask-angle | Length     |                  |
| mask-image | Same as `background-image` |                  |
| mask-mode | luminance / alpha |                  |
| mask-position | Length, Length (optional) |                  |
| mask-position-x | Length     |                  |
| mask-position-y | Length     |                  |
| mask-repeat | same as `background-repeat` |                  |
| mask-size | Length, Length (optional) |                  |
| mask-size-x | Length     |                  |
| mask-size-y | Length     |                  |
| max-height | Length     |                  |
| max-width | Length     |                  |
| min-height | Length     |                  |
| min-width | Length     |                  |
| mix-blend-mode | normal / lighten / multiply |                  |
| opacity | Float      |                  |
| order | int        |                  |
| overflow | visible (default) / hidden / scroll |                  |
| overflow-x | Same as `overflow` |                  |
| overflow-y | Same as `overflow` |                  |
| padding | Fills in the properties below |                  |
| padding-bottom | Length     |                  |
| padding-left | Length     |                  |
| padding-right | Length     |                  |
| padding-top | Length     |                  |
| perspective-origin | Length, Length (optional) |                  |
| perspective-origin-x | Length     |                  |
| perspective-origin-y | Length     |                  |
| pointer-events | none (default) / all / auto |                  |
| **position\*** | static (default) / relative / absolute | See how it works: <https://yogalayout.com/docs/absolute-relative-layout/> |
| right | Length     |                  |
| row-gap | Length     |                  |
| text-align | left (default) / center / right |                  |
| text-background-angle | Length     |                  |
| text-decoration | Color / Length / LineStyle, Line | Properties can be in any order and you can have multiple lines. |
| text-decoration-color | Color      |                  |
| text-decoration-line | underline / line-through / overline | Multiple properties can be set. eg. `text-decoration-line: overline underline;` |
| text-decoration-line-through-offset | Length     |                  |
| text-decoration-overline-offset | Length     |                  |
| text-decoration-skip-ink | all / none | Decides whether the line decoration should draw above glyphs or not |
| text-decoration-thickness | Length     |                  |
| text-decoration-underline-offset | Length     |                  |
| text-overflow | clip / ellipsis |                  |
| text-shadow | Same as `box-shadow` |                  |
| text-transform | none (default) / capitalize / lowercase / uppercase |                  |
| top  | Length     |                  |
| transform | Fills in the properties below |                  |
| transform-origin | Length, Length, Length (optional) |                  |
| transform-origin-x | Length     |                  |
| transform-origin-y | Length     |                  |
| transition | Fills in the properties below | `transition: all 0.1s ease;`,`transition: opacity 0.1s transform 0.2s ease-out;` |
| transition-delay | Float      |                  |
| transition-duration | Float      |                  |
| transition-property | String     |                  |
| transition-timing-function | linear (default) / ease / ease-in-out / ease-out / ease-in |                  |
| white-space | normal / nowrap / pre | Use `pre` to format tabs and newlines. |
| width | Length     |                  |
| word-break | normal / break-all |                  |
| word-spacing | Length     |                  |
| z-index | int        |                  |

## Custom Pseudo-Classes

These are some s&box-specific pseudo-classes which are useful for applying transitions when an element is created or deleted.

* `:intro` is removed when the element is created, things will transition away from this
* `:outro` is added when Panel.Delete() is called. The Panel waits for all transitions to finish before actually deleting itself.

```scss
MyPanel {
	transition: all 2s ease-out;
	transform: scale( 1 );

	// When the element is created make it expand from nothing.
	&:intro {
		transform: scale( 0 );
	}

	// When the element is deleted make it double in size before being deleted.
	&:outro {
		transform: scale( 2 );
	}
}
```
