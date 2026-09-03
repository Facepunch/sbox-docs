---
title: "Style Properties"
icon: "🎨"
created: 2024-09-24
updated: 2026-09-03
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

## Value Syntax & Units

These work across most properties:

| Feature | Description | Examples |
|---------|-------------|----------|
| `min()` / `max()` / `clamp()` | Math functions usable anywhere a `Length` is accepted. | `width: min(100px, 50%);`,`width: clamp(10px, 50%, 200px);` |
| CSS-wide keywords | `inherit`, `initial`, `unset` and `revert` are accepted on single-value properties and most shorthands (`margin`, `border`, `background`, `filter`, `animation`, …). Not on `box-shadow`, `text-shadow`, `filter-drop-shadow`, `background-image`, `mask-image`, `border-image`, `transition*` or `flex`. | `color: inherit;`,`margin: unset;` |
| `currentColor` | Resolves to the element's current `color` value. | `border-color: currentColor;` |
| `oklch()` / `lab()` / `hwb()` | Additional color syntaxes, on top of hex / `rgb()` / `rgba()`. | `color: oklch(0.7 0.15 200);` |
| Viewport units | `dvh`, `svh`, `lvh` and `dvw` are treated the same as `vh` / `vw`. | `height: 100dvh;` |
| Time units | Durations accept `ms` as well as `s`. | `transition: opacity 200ms ease;`,`animation-duration: 500ms;` |

## Custom Style Properties

| Name | Parameters | Examples / Notes |
|------|------------|------------------|
| aspect-ratio | Float, Float (Optional) / auto | `aspect-ratio: 1;`,`aspect-ratio: 16/9;`,`aspect-ratio: auto;` |
| background-image-tint | Color      | Multiplies the `background-image` by this Color. Not a replacement for `filter` or `backdrop-filter`. |
| border-image-tint | Color      | Multiplies the `border-image` by this Color. |
| border-shape | none / polygon(x y, ...) / circle(radius at x y) | Replaces the rounded rectangle with a polygon (3-8 points) or a circle, for backgrounds, borders and hit testing. See [Border Shape](border-shape.md). |
| mask-scope | default / filter | `default` will apply the mask normally, `filter` will use the mask to blend between unfiltered and filtered. |
| sound-in | String     | The name of the sound to play when this style is applied to an element. This is useful to put on a `:hover` or `:active` style to play a sound on hover/click |
| sound-out | String     | The name of a sound to play when this style is removed from an element. |
| text-stroke | Length, Color | This will put an outline |
| text-stroke-color | Color      |                  |
| text-stroke-width | Length     |                  |

## Supported Style Properties

| Name | Parameters | Examples / Notes |
|------|------------|------------------|
| align-content | auto / flex-start / flex-end / center / stretch / space-between / space-around / start / end / baseline |                  |
| align-items | Same as `align-content` |                  |
| align-self | Same as `align-content` |                  |
| animation | Fills in the properties below |                  |
| animation-delay | Time (s / ms) | A unit is required. eg. `animation-delay: 200ms;` |
| animation-direction | normal (default) / reverse / alternate / alternate-reverse |                  |
| animation-duration | Time (s / ms) | A unit is required. eg. `animation-duration: 0.5s;` |
| animation-fill-mode | none (default) / forwards / backwards / both |                  |
| animation-iteration-count | int / infinite |                  |
| animation-name | String     |                  |
| animation-play-state | running (default) / paused |                  |
| animation-timing-function | Same as `transition-timing-function` |                  |
| backdrop-filter | blur(Length) <br>saturate(Length) <br>contrast(Length) <br>brightness(Length) <br>grayscale(Length) <br>sepia(Length) <br>hue-rotate(Length) <br>invert(Length) / none | `backdrop-filter: blur(10px) saturate(80%);` `none` clears any backdrop filters. |
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
| border-width | Length (1–4 values) | Accepts 1–4 values like the CSS shorthand. eg. `border-width: 1px;`,`border-width: 1px 2px;`,`border-width: 1px 2px 3px 4px;` |
| bottom | Length     |                  |
| box-shadow | Length (x),<br>Length (y),<br>Length (blur, optional),<br>Length (spread, optional),<br>Color | `box-shadow: 2px 2px 4px black;` |
| caret-color | Color      | The text cursor in a `TextEntry`. Defaults to the text colour. |
| color | Color /<br>linear-gradient(Color, Color) /<br>radial-gradient(Color, Color) |                  |
| column-gap | Length     |                  |
| content | string     | Sets the text of a Label.<br>eg. `content: "Loading…";` |
| cursor | none / pointer / progress / wait / crosshair / text / move / not-allowed / any custom cursors |                  |
| **display\*** | flex (default) / none / contents | Everything is flex by default |
| filter | Same as `backdrop-filter`, plus <br>tint(Color) <br>drop-shadow(Same as `box-shadow`) <br>border-wrap(Length, Color) / none | `filter: blur(2px) tint(#f00);` `none` clears any filters. |
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
| filter-tint | Color      |                  |
| flex-basis | Length     |                  |
| flex-direction | row (default) / row-reverse / column / column-reverse |                  |
| flex-flow | flex-direction, flex-wrap | Shorthand for `flex-direction` and `flex-wrap`. eg. `flex-flow: row wrap;` |
| flex-grow | Float      |                  |
| flex-shrink | Float      |                  |
| flex-wrap | nowrap (Default) / wrap / wrap-reverse |                  |
| font | Fills in the font properties below | Shorthand for `font-style`, `font-weight`, `font-size`, `line-height` and `font-family`. eg. `font: italic bold 16px/1.4 Poppins;` |
| font-color | Color      |                  |
| **font-family\*** | String     | Specify a single font, based on the name of the font itself, not the filename.<br>eg. `font-family: Comic Sans MS;`<br>Generic families (`serif`, `sans-serif`, `monospace`) are mapped to a default font. |
| font-size | Length / xx-small / x-small / small / medium / large / x-large / xx-large / xxx-large |                  |
| font-smooth | auto / always / never / none | `never` is good for pixel fonts |
| font-style | normal  (default) / italic |                  |
| font-variant-numeric | normal / tabular-nums |                  |
| font-weight | normal (default) / bold / light / bolder / lighter / black / int | `font-weight: bold;`,`font-weight: 300;` |
| gap  | Length, Length (optional) / normal | Shorthand for `row-gap` and `column-gap`, specified the size of gutters. |
| height | Length     |                  |
| image-rendering | auto (default) / anisotropic / bilinear / trilinear / point / pixelated / nearest-neighbor / crisp-edges | `crisp-edges` uses point sampling. |
| inset | Length (1–4 values) | Shorthand for `top`, `right`, `bottom` and `left`. |
| inset-block | Length, Length (optional) | Shorthand for `inset-block-start` and `inset-block-end`. |
| inset-block-end | Length     |                  |
| inset-block-start | Length     |                  |
| inset-inline | Length, Length (optional) | Shorthand for `inset-inline-start` and `inset-inline-end`. |
| inset-inline-end | Length     |                  |
| inset-inline-start | Length     |                  |
| justify-content | Same as `align-content` |                  |
| left | Length     |                  |
| letter-spacing | Length / normal |                  |
| line-height | Length     |                  |
| margin | Fills in the properties below |                  |
| margin-block | Length, Length (optional) | Shorthand for `margin-block-start` and `margin-block-end`. |
| margin-block-end | Length     |                  |
| margin-block-start | Length     |                  |
| margin-bottom | Length     |                  |
| margin-inline | Length, Length (optional) | Shorthand for `margin-inline-start` and `margin-inline-end`. |
| margin-inline-end | Length     |                  |
| margin-inline-start | Length     |                  |
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
| object-fit | fill / contain / cover (default) / none / scale-down | `scale-down` is treated as `contain`. |
| opacity | Float / Percentage | `opacity: 0.5;`,`opacity: 50%;` |
| order | int        |                  |
| overflow | visible (default) / hidden / scroll / auto | `auto` maps to `scroll`. |
| overflow-x | Same as `overflow` |                  |
| overflow-y | Same as `overflow` |                  |
| padding | Fills in the properties below |                  |
| padding-block | Length, Length (optional) | Shorthand for `padding-block-start` and `padding-block-end`. |
| padding-block-end | Length     |                  |
| padding-block-start | Length     |                  |
| padding-bottom | Length     |                  |
| padding-inline | Length, Length (optional) | Shorthand for `padding-inline-start` and `padding-inline-end`. |
| padding-inline-end | Length     |                  |
| padding-inline-start | Length     |                  |
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
| **scrollbar-color\*** | Color, Color (optional) / auto | Thumb colour then track colour. `auto` clears both. Inherited. |
| scrollbar-gutter | auto (default) / stable / stable both-edges | `stable` reserves the vertical bar's width so content is laid out beside it instead of under it. |
| **scrollbar-width\*** | auto / thin / none (default) / Length | Zero or `none` means no bar. Unlike the web it's inherited, so one rule on a root turns bars on for a whole tree, and a child can opt out with `none`. |
| text-align | left (default) / center / right / justify / start / end |                  |
| text-background-angle | Length     |                  |
| text-decoration | Color / Length / LineStyle, Line | Properties can be in any order and you can have multiple lines. |
| text-decoration-color | Color      |                  |
| text-decoration-line | none / underline / line-through / overline | Multiple properties can be set. eg. `text-decoration-line: overline underline;` |
| text-decoration-skip-ink | auto / all / none | Decides whether the line decoration should draw above glyphs or not |
| text-decoration-style | solid (default) / double / dotted / dashed / wavy |                  |
| text-decoration-thickness | Length     |                  |
| text-line-through-offset | Length     |                  |
| text-overflow | clip / ellipsis |                  |
| text-shadow | Same as `box-shadow` |                  |
| text-transform | none (default) / capitalize / lowercase / uppercase |                  |
| top  | Length     |                  |
| transform | Fills in the properties below | `scale()` accepts comma-separated arguments. eg. `transform: scale(2, 0.5);` |
| transform-origin | Length, Length (optional) |                  |
| transform-origin-x | Length     |                  |
| transform-origin-y | Length     |                  |
| transition | Fills in the properties below | `transition: all 0.1s ease;`,`transition: opacity 0.1s, transform 0.2s ease-out;` |
| text-overline-offset | Length     |                  |
| transition-delay | Time (s / ms) | A unit is required. eg. `transition-delay: 200ms;` |
| transition-duration | Time (s / ms) | A unit is required. eg. `transition-duration: 0.5s;` |
| text-underline-offset | Length     |                  |
| transition-property | String     |                  |
| transition-timing-function | ease (default) / linear / ease-in / ease-out / ease-in-out / bounce-in / bounce-out / bounce-in-out / sin-ease-in / sin-ease-out / sin-ease-in-out / step-start / step-end / cubic-bezier(x1, y1, x2, y2) / steps(n, start / end) | Use `cubic-bezier()` and `steps(n, end)` in the `transition` / `animation` shorthands or `animation-timing-function`; this longhand splits its value on commas, so they don't parse here. |
| white-space | normal / nowrap / pre / pre-wrap / pre-line / break-spaces | Use `pre` to format tabs and newlines. |
| width | Length     |                  |
| word-break | normal / break-all / break-word |                  |
| word-spacing | Length / normal |                  |
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
