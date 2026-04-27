---
title: Base Addon Code Architecture
sources:
  - game/addons/base/code/
created: 2026-04-27
updated: 2026-04-27
---

# Base Addon Code Architecture

The `game/addons/base/code/` directory contains the foundational C# logic, UI controls, and utility systems that most s&box projects rely on. It acts as the standard library for game development in s&box.

## Quick Working Example

You can explore the source code of standard UI controls to learn how they are implemented and create your own variations.

```csharp
using Sandbox.UI;

// Example of a basic UI control similar to those found in base/code/UI/Controls/
public class CustomButton : Button
{
    public CustomButton()
    {
        AddClass( "custom-button" );
    }

    protected override void OnClick( MousePanelEvent e )
    {
        base.OnClick( e );
        Log.Info( "Custom Button Clicked!" );
    }
}
```

1. **Studying UI Implementation:** If you want to build robust, reusable Razor components, studying the `UI/Controls/` directory is highly recommended. It demonstrates best practices for state management, events, and styling within the s&box UI framework.
2. **Citizen Animations:** The `CitizenAnimationHelper` is the standard way to drive the complex animation graph of the Citizen model. It abstracts away the raw parameter setting into easy-to-use C# methods.

- **Overriding Base Assets:** Remember that you can override assets (like materials or sounds) provided by the base addon by placing a file with the identical path in your own project.

## Troubleshooting

:::warning Common Gotchas
- **Missing References:** If you cannot access classes like `Button` or `TextEntry`, ensure your `.sbproj` file includes `facepunch.base` in its dependencies and that you have `using Sandbox.UI;` at the top of your file.
:::

## Related Pages
- [Base Addon Overview](./base.md)

