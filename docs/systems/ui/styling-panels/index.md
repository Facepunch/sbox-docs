---
title: "Styling Panels"
icon: "🖌️"
created: 2024-09-24
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Systems/UI/Styles/StyleSheet.cs
  - engine/Sandbox.Engine/Systems/UI/Styles/BaseStyles.cs
---

# Styling Panels

> **New to UI?** Reference for styling Razor panels; UI overview is at **[UI](../index.md)**.

s&box uses a CSS-like styling system and Flexbox layout engine to give your panels visual appeal, using either external `.scss` files or inline C#.

## Quick Working Example

If you have a Razor panel named `Health.razor`, simply create a `Health.razor.scss` file in the same folder. The engine will automatically compile and apply it to your panel.

**Health.razor.scss:**
```css
Health {
    width: 200px;
    height: 50px;
    background-color: rgba(black, 0.5);
    border-radius: 8px;
    padding: 8px;

    .hp-bar {
        height: 100%;
        background-color: red;
        transition: width 0.2s ease;
    }
}
```

**Health.razor:**
```csharp
@using Sandbox;
@using Sandbox.UI;
@inherits PanelComponent

<root>
    <div class="hp-bar" style="width: @(HealthRatio * 100f)%"></div>
</root>

@code {
    public float HealthRatio { get; set; } = 0.75f;
    protected override int BuildHash() => System.HashCode.Combine( HealthRatio );
}
```

## Navigation Hub
- [**Style Properties**](style-properties.md): A comprehensive list of supported CSS properties and s&box specific custom properties.

### Linking Custom Stylesheets

While `.razor.scss` files load automatically, you can also manually link shared stylesheets using an attribute on your C# class.

```csharp
[StyleSheet("/ui/global.scss")]
public sealed class MainMenu : PanelComponent
{
    // ...
}
```

### SCSS Imports

You can organize your styles by importing common variables or mixins into other stylesheets.

```css
@import "/ui/theme.scss";

.my-button {
    background-color: $primary-color;
}
```

### Inline Styling (In Layout)

For dynamic values (like a progress bar's width), it is usually easier to use inline styles within your Razor markup. You can seamlessly inject C# variables using the `@` symbol.

```markup
<div class="progress-bar">
    <div class="fill" style="width: @(Progress * 100f)%"></div>
</div>
```

### Inline Styling (In Code)

You can also modify a panel's style directly from C#. This is useful when you are building panels entirely via code instead of using Razor templates.

```csharp
var myPanel = new Panel();
myPanel.Parent = this.Panel;
myPanel.Style.Width = Length.Pixels( 200 );
myPanel.Style.BackgroundColor = Color.Red;
```

## Troubleshooting

:::danger "My SCSS styles aren't updating!"
If you rename a component or its SCSS file, ensure they still follow the strict naming convention: `MyComponent.razor` must pair with `MyComponent.razor.scss`.
:::

:::warning "Wait, why doesn't `float: left` work?"
s&box uses a Flexbox-based layout system. Properties like `float` or CSS Grid are not supported. Use `flex-direction`, `justify-content`, and `align-items` instead.
:::

## Related Pages
- [Razor Panels](../razor-panels/index.md)
- [Style Properties](style-properties.md)
