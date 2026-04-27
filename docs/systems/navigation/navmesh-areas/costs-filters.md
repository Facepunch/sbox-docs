---
title: "Costs & Filters"
icon: "🚦"
created: 2025-10-22
updated: 2026-04-12
sources:
  - engine/Sandbox.Engine/Resources/NavMeshAreaDefinition.cs
  - engine/Sandbox.Engine/Scene/Components/Navigation/NavMeshArea.cs
  - engine/Sandbox.Engine/Scene/Components/Navigation/NavMeshAgent.cs
---

# Costs & Filters

Instead of completely blocking a path, you can use `NavMeshAreaDefinition` resources to mark certain areas as difficult to traverse, or restrict access to only certain agents.

## Quick Working Example

You can assign an area definition to a `NavMeshArea` in the editor, or via code.

```csharp
public class SwampController : Component
{
    [Property] public NavMeshArea SwampArea { get; set; }
    [Property] public NavMeshAreaDefinition SwampDefinition { get; set; }

    protected override void OnStart()
    {
        // Apply the swamp rules (e.g. higher cost to traverse) to this area
        SwampArea.IsBlocker = false;
        SwampArea.Area = SwampDefinition;
    }
}
```

### Creating a Custom Area Definition

1. In the Editor Assets Browser, right-click and select **Create -> NavArea Definition**.
2. Name it (e.g., `ShallowWater.navarea`).
3. Select the new file. In the Inspector, you can configure its settings.

### Configuration

| Property | Description |
|---|---|
| **Color** | The color used to render this area in the editor when viewing the NavMesh. |
| **CostMultiplier** | How much costlier it is to cross this Area (from `1.0` to `100.0`). A value of `2.0` means it's twice as hard to walk here. |
| **Priority** | If two `NavMeshArea` volumes overlap in the scene, the one with the higher Priority value will dictate the rules for the overlapping section. |

### Filters

You can specify which areas an agent is allowed to traverse. For example, you might create a `SmallDoorway.navarea` definition. Normal agents are blocked by it, but "Shortcutter" agents are allowed to use it. You can define what areas an agent can traverse using its properties.

```csharp
public class SpecialAgent : Component
{
    [Property] public NavMeshAgent Agent { get; set; }
    [Property] public NavMeshAreaDefinition DoorwayDefinition { get; set; }

    protected override void OnStart()
    {
        // Only allow this agent to traverse the "Doorway" area
        Agent.AllowedAreas.Add( DoorwayDefinition );

        // Conversely, forbid the agent from traversing the Doorway area
        // Agent.ForbiddenAreas.Add( DoorwayDefinition );

        // Prevent the agent from moving on the default navigation mesh area
        // Agent.AllowDefaultArea = false;
    }
}
```

#### Filter Configuration

| Property | Description |
|---|---|
| `AllowedAreas` | A `HashSet<NavMeshAreaDefinition>` specifying which areas the agent is allowed to travel on. If empty, all areas are allowed. |
| `ForbiddenAreas` | A `HashSet<NavMeshAreaDefinition>` specifying which areas the agent is *not* allowed to travel on. Takes precedence over allowed areas. |
| `AllowDefaultArea` | A `bool` (default `true`) indicating whether the agent is allowed to travel on areas that do not have a specific definition assigned. |

## Troubleshooting

:::danger Agents ignoring the cost?
Ensure your `NavMeshArea` has **Is Blocker** set to `false`. If it is true, it acts as an obstacle and the `Area` definition is ignored. Also, ensure your `CostMultiplier` is high enough. If the detour around the swamp is extremely long, the agent may still decide the swamp is the mathematically fastest route.
:::

## Related Pages
- [NavMesh Areas Overview](index.md)