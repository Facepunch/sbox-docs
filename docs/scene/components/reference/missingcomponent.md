---
title: "Missing Component"
icon: "❌"
sources:
  - engine/Sandbox.Engine/Scene/Components/MissingComponent.cs
updated: 2026-04-25
created: 2026-04-27
---

# Missing Component

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `MissingComponent` component.

The `MissingComponent` acts as a placeholder when a scene fails to deserialize a component, ensuring its JSON data is preserved and not lost.

## Quick Working Example

```csharp
using Sandbox;
using System.Text.Json.Nodes;

public sealed class MissingComponentScanner : Component
{
	protected override void OnStart()
	{
		// Find all missing components in the scene
		var missingComponents = Scene.GetAllComponents<MissingComponent>();

		foreach ( var missing in missingComponents )
		{
			// Extract the original JSON data to log or analyze it
			JsonObject originalData = missing.GetJson();
			Log.Warning( $"Found a missing component on {missing.GameObject.Name}. Original data: {originalData}" );
		}
	}
}
```

### Retrieving Lost Data

If you need to programmatically recover data from a missing component (for example, you are writing an editor tool to migrate old component data to a new format), you can use the `GetJson()` method.

```csharp
var missing = myGameObject.GetComponent<MissingComponent>();
if ( missing != null )
{
    var json = missing.GetJson();
    
    // Check if the old component had a specific property
    if ( json.TryGetPropertyValue("OldSpeedLimit", out var speedNode) )
    {
        float speed = speedNode.GetValue<float>();
        Log.Info( $"Recovered old speed limit: {speed}" );
    }
}
```

## Configuration

The `MissingComponent` has no public properties visible in the Inspector besides the indication that an error occurred. It uses `ComponentFlags.Error` to visually flag itself in the editor.

## Troubleshooting

:::danger "Component is Missing" Error in Inspector
If you open a scene and see a red "Missing Component" warning on a GameObject, it means the C# class that originally lived there no longer exists or failed to compile.

**How to fix:**
1. Check your console for C# compiler errors. Fix those first!
2. Did you rename a component class? If so, the scene is still looking for the old name.
3. Did you delete the script entirely? If you no longer need the component, you can safely remove the `MissingComponent` from the GameObject to clean it up.
:::

## Related Pages
- [Components Overview](../index.md)
