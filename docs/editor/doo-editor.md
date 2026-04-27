---
title: "Doo Editor"
icon: "🌲"
sources:
  - game/editor/DooEditor/Code/DooEditor/DooEditorWidget.cs
updated: 2026-04-25
created: 2026-04-27
---

# Doo Editor

The Doo Editor is an internal tool for visually scripting or configuring `Doo` blocks.

## Quick Working Example

If a property in your C# code is of type `Doo`, the inspector will display a custom control widget that allows you to open the Doo Editor. 

```csharp
using Sandbox;

public sealed class MyInteractiveComponent : Component
{
    [Property]
    public Doo OnInteract { get; set; }

    public void TriggerInteraction()
    {
        // Execute the visual script configured in the Doo Editor
        OnInteract?.Execute();
    }
}
```

When you open the Doo Editor, you'll see a toolbox of available blocks:
* **Invoke:** Calls a specific method.
* **Set:** Sets a value on a property.
* **Delay:** Waits for a specific amount of time before continuing execution.
* **For:** Loops over a block of logic.

You can drag these from the toolbox into your block tree to compose logic. The Editor will automatically provide the `DooControlWidget` in the inspector, allowing users to configure the `OnInteract` logic without writing code.

## Performance

- **Execution Overhead:** Similar to ActionGraph, executing logic defined by `Doo` blocks has slightly more overhead than running compiled C# code. Avoid using `Doo` blocks within tight, performance-critical loops like `OnUpdate`. Use them for higher-level sequence control, such as triggering events or coordinating UI changes.
- **Serialization Size:** Since `Doo` logic is saved within the Component's property data, extensive visual scripts can increase the size of the saved scene or prefab file. Keep `Doo` scripts relatively short and focused.

- **Missing Method Exceptions:** If a method referenced by an `Invoke` block is renamed, removed, or has its parameters changed in C# after the `Doo` script was configured, the `Invoke` block will fail at runtime. You must re-configure the block in the Doo Editor to point to the correct method.

## Troubleshooting

:::warning "Doo Script Not Running"
Ensure that your C# component is actually calling `Execute()` on the `Doo` property, and that the `GameObject` the component is attached to is active. If `OnInteract?.Execute()` isn't reached, the visual script won't run.
:::

## Sample Projects

*   [**Sweeper**](../build-games/samples-templates/sweeper.md) - Contains examples of simple sequence triggering that could be managed via Doo blocks.

## Related Pages
* [ActionGraph](../systems/actiongraph/index.md)
* [Property Attributes](property-attributes.md)
