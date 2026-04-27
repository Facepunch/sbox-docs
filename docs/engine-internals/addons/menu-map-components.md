---
title: Menu Map Components
icon: "🗺️"
sources:
  - game/addons/menu/Code/Map/Button.cs
  - game/addons/menu/Code/Map/LinearMove.cs
  - game/addons/menu/Code/Map/CursorPresser.cs
created: 2026-04-27
updated: 2026-04-27
---

# Menu Map Components

The `game/addons/menu/Code/Map/` directory contains components meant to facilitate basic gameplay interactions in the background map that loads behind the main menu. These are generic interactivity components useful for building interactive UI background environments.

## Quick Working Example

You can use `CursorPresser` paired with a `Button` to allow players to click on interactive objects in the 3D menu background.

```csharp
using Sandbox;
using Sandbox.Mapping;

public sealed class MyMenuInteractor : Component
{
    protected override void OnStart()
    {
        // Add a CursorPresser to your menu camera to enable clicking on IPressable components
        var presser = Components.GetOrCreate<CursorPresser>();
    }
}
```

## Common Components

- **`Button`:** A generic interaction point. It handles playing sounds, firing `Action` delegates, and tracking whether it acts as a toggle, a continuous press, or a single press mechanism.
- **`LinearMove`:** Often paired with a `Button` to act as a door or sliding platform. It uses a `Curve` to define its motion path over time between two points when triggered.

## Troubleshooting

:::warning
- **Cursor clicks not registering:** Ensure the `GameObject` with the `Button` or `LinearMove` component also has a `Collider` (like a `BoxCollider`). `CursorPresser` relies on `Scene.Trace` to find interactive objects.
:::

## Related Pages
- [Main Menu Addon](index.md)

