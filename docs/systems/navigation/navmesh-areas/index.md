---
title: "NavMesh Areas"
icon: "⏹️"
created: 2024-12-03
updated: 2026-04-12
sources:
  - engine/Sandbox.Engine/Scene/Components/Navigation/NavMeshArea.cs
  - engine/Sandbox.Engine/Resources/NavMeshAreaDefinition.cs
---

# NavMesh Areas

`NavMeshArea` components allow you to define specific zones on the NavMesh to block pathing, increase movement cost, or restrict access for certain agents.

## Quick Working Example

You can dynamically block off an area of the NavMesh at runtime by adding a `NavMeshArea` to a volume.

```csharp
public class BridgeController : Component
{
	[Property] public NavMeshArea BridgeArea { get; set; }

	public void CloseBridge()
	{
		// Treat this area as an obstacle, recalculating the NavMesh locally
		BridgeArea.IsBlocker = true;
	}

	public void OpenBridge()
	{
		// Allow agents to walk here again
		BridgeArea.IsBlocker = false;
	}
}
```

## Navigation Hub

Explore how to customize your NavMesh areas:

* [**Obstacles**](obstacles.md) - Completely block off sections of the NavMesh from generation.
* [**Costs & Filters**](costs-filters.md) - Make areas more "expensive" to walk through so agents prefer other paths, or restrict areas to specific types of agents.

## Limitations & Best Practices

* **Static areas** are almost free computationally. They are baked directly into the NavMesh.
* **Moving areas** (like an obstacle attached to a moving vehicle) have a performance cost because the NavMesh must be rebuilt locally around them as they move. You can safely have a few dozen moving areas, but avoid using them on hundreds of objects.
* Currently, there is a limit of 24 unique `NavMeshAreaDefinition` resources per project, but you can assign those definitions to as many `NavMeshArea` components as you like.
