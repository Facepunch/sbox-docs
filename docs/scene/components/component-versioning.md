---
title: "Component Versioning"
icon: "🔢"
created: 2024-01-09
updated: 2024-11-07
sources:
  - engine/Sandbox.Engine/Scene/Components/Component.cs
---

# Component Versioning

Component versioning allows you to safely upgrade existing components in a scene or prefab when you make breaking changes to their properties, such as renaming fields or changing data types.

## Quick Working Example

```csharp
public sealed class MyComponent : Component
{
    // The current version of this component
    public override int ComponentVersion => 2;

    // The new property we want to use
    [Property] public string[] StringPropertyArray { get; set; }

    /// <summary>
    /// Upgrades from Version 0/1 to Version 2.
    /// Replaces the old 'StringProperty' string with a new string array.
    /// </summary>
    [JsonUpgrader( typeof( MyComponent ), 2 )]
    private static void UpgradeToStringArray( JsonObject json )
    {
        // 1. Check if the old property exists
        if ( json.Remove( "StringProperty", out var oldNode ) )
        {
            // 2. Create a new array and put the old value inside it
            var jsonArray = new JsonArray { oldNode };

            // 3. Assign the new array to the new property name
            json["StringPropertyArray"] = jsonArray;
        }
    }
}
```

### Upgrading Incrementally

Upgrades are chained. If a component in a scene is currently at Version 0, and your code defines `ComponentVersion => 2` with upgraders for version 1 and 2, the engine will run both upgraders in sequence.

```csharp
public sealed class MyComponent : Component
{
    public override int ComponentVersion => 2;
    
    [Property] public string NewStringProperty { get; set; }
    
    // Upgrade Version 0 -> Version 1
    [JsonUpgrader( typeof( MyComponent ), 1 )]
    private static void UpgradeVersion1( JsonObject json )
    {
        // Make some changes
    }
    
    // Upgrade Version 1 -> Version 2
    [JsonUpgrader( typeof( MyComponent ), 2 )]
    private static void UpgradeVersion2( JsonObject json )
    {
        // Make further changes
    }
}
```

### Renaming a Property

The most common use case is simply renaming a property.

```csharp
[JsonUpgrader( typeof( MyComponent ), 1 )]
private static void RenameProperty( JsonObject json )
{
    // Remove the old property value
    if ( json.Remove( "OldPropertyName", out var oldNode ) )
    {
        // Re-add it under the new name
        json["NewPropertyName"] = oldNode;
    }
}
```

## Configuration

| Property/Attribute | Description |
|---|---|
| `ComponentVersion` | A virtual property on `Component`. Override it to return the current version integer (default is 0). |
| `[JsonUpgrader]` | Attribute applied to a `static` method to define it as an upgrader. Takes the component type and the version number it upgrades *to*. |

## Troubleshooting

:::danger Forgotten ComponentVersion
If you write a `[JsonUpgrader]` but forget to override `ComponentVersion` to match the target version, your upgrader will never run, and your data will be lost! Always double-check that `ComponentVersion => X` matches `[JsonUpgrader( ..., X )]`.
:::

:::warning Upgrader Method Signature
Your upgrader method **must** be `static` and take exactly one argument of type `JsonObject`. If the signature is incorrect, the engine will not execute it.
:::

## Related Pages
* [Component Lifecycle](component-lifecycle.md)
* [Components Overview](../index.md)
