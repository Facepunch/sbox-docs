---
title: "Create a Component"
icon: "🧩"
sources:
  - engine/Sandbox.Engine/Scene/Components/Component.cs
created: 2026-04-27
updated: 2026-04-27
---

# Create a Component

Components are the primary way to add custom behavior and functionality to GameObjects.

## Quick Working Example

Create a new file named `MyCustomLogic.cs` in your `Assets` directory with the following structure:

```csharp
using Sandbox;

public sealed class MyCustomLogic : Component
{
    // A property that will show up in the Editor Inspector
    [Property] public float Speed { get; set; } = 100f;

    // Called once when the component is enabled
    protected override void OnStart()
    {
        Log.Info( "Component started!" );
    }

    // Called every frame
    protected override void OnUpdate()
    {
        GameObject.WorldPosition += Vector3.Forward * Speed * Time.Delta;
    }
}
```

### Basic Implementation

1. **Inheritance:** Your class must inherit from `Component`.
2. **Sealed:** It is best practice to mark components as `sealed` unless you intend to inherit from them.
3. **Properties:** Add the `[Property]` attribute to any variables you want to tweak from the Editor.
4. **Lifecycle Hooks:** Override methods like `OnStart` or `OnUpdate` to inject your game logic at specific times.

Once saved, the engine will instantly hot-load your script. You can then select any GameObject in the scene, click **Add Component** in the Inspector, and search for `MyCustomLogic`.

### Editor Only Components

If you want a component to only run logic inside the editor (like generating a procedural mesh or snapping objects to a grid), add the `Component.ExecuteInEditor` interface to the class.

```csharp
public sealed class GridSnapper : Component, Component.ExecuteInEditor
{
    protected override void OnUpdate()
    {
        // This will run while you are moving objects in the Editor viewport
    }
}
```

## Configuration

| Property | Description |
|:---|:---|
| `Enabled` | Gets or sets whether this component is enabled. Disabled components will not have their update loop called. |

## Troubleshooting

:::warning "My component isn't showing up in the 'Add Component' list!"
Ensure you have saved the C# file and that there are no compilation errors in your project. Check the Editor Console for any red error messages. If your code does not compile, new components will not appear.
:::

:::danger "Null Reference in OnAwake"
If you override `OnAwake` and try to access other components on the GameObject, they might not be initialized yet. Use `OnStart` for logic that relies on other components being ready.
:::

## Related Pages
- [Component Lifecycle](../scene/components/component-lifecycle.md)
- [Your First Component](../tutorials/your-first-component.md)
