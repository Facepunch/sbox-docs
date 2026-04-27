---
title: "NavMesh Agent"
icon: "🤖"
sources:
  - engine/Sandbox.Engine/Scene/Components/Navigation/NavMeshAgent.cs
updated: 2026-04-25
created: 2026-04-27
---

# NavMesh Agent

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `NavMeshAgent` component.

`NavMeshAgent` is an autonomous agent that navigates the scene's navmesh. It plans paths, avoids other agents (crowd separation), and traverses off-mesh links. The agent owns its position internally (via DotRecast Crowd) and, by default, drives the GameObject's `Position` to follow.

## Quick Working Example

```csharp
public class Bot : Component
{
    [RequireComponent] public NavMeshAgent Agent { get; set; }

    [Property] public GameObject Target { get; set; }

    protected override void OnUpdate()
    {
        if ( Target.IsValid() )
            Agent.MoveTo( Target.WorldPosition );
    }
}
```

## Common Patterns

### Spam-safe MoveTo

`MoveTo` checks the current target and only re-requests when the target actually changes. You can call it every frame without trashing the path planner.

```csharp
Agent.MoveTo( player.WorldPosition );
```

### Driving your own controller

Read `WishVelocity` and feed it into a `CharacterController` or `PlayerController` instead of letting the agent move the transform.

```csharp
[RequireComponent] NavMeshAgent Agent { get; set; }
[RequireComponent] CharacterController Controller { get; set; }

protected override void OnUpdate()
{
    Agent.UpdatePosition = false;
    Agent.MoveTo( Target );

    Controller.Velocity = Agent.WishVelocity;
    Controller.Move();

    // Keep agent in sync with where the controller actually ended up
    Agent.SetAgentPosition( WorldPosition );
}
```

### Custom link traversal

If `AutoTraverseLinks` is `false`, you handle off-mesh link traversal yourself. Listen to `LinkEnter` / `LinkExit`, animate the GameObject between `CurrentLinkTraversal.LinkEnterPosition` and `LinkExitPosition`, then call `CompleteLinkTraversal()`.

### Reading the path

`GetPath()` returns the current `NavMeshPath` (it's not free — don't call it every frame). Use `IsNavigating` to check whether a target is actively being pursued.

## Properties

| Property | Default | Description |
|---|---|---|
| `Height` | `64` | Cylinder height used for the agent in the navmesh crowd. |
| `Radius` | `16` | Cylinder radius used for crowd avoidance and collision queries. |
| `MaxSpeed` | `120` | Maximum movement speed (units/second). |
| `Acceleration` | `120` | How fast the agent can change velocity. Set >= `MaxSpeed` for snappy stops/turns. |
| `UpdatePosition` | `true` | If true, the GameObject's position is set to `AgentPosition` each frame. |
| `UpdateRotation` | `false` | If true, the GameObject is rotated to face its lookahead direction. |
| `AllowedAreas` | `new()` | Set of `NavMeshAreaDefinition`s the agent may traverse. Empty means "all". |
| `ForbiddenAreas` | `new()` | Set of `NavMeshAreaDefinition`s the agent must avoid. |
| `AllowDefaultArea` | `true` | Whether the agent can walk on the default (untagged) area. |
| `AutoTraverseLinks` | `true` | If true, the crowd traverses off-mesh links automatically. |
| `Separation` | `0.25` | 0–1 weight controlling avoidance/spread between nearby agents. |

## Read-only state

| Property | Description |
|---|---|
| `AgentPosition` | The crowd's authoritative position, even if `UpdatePosition` is false. |
| `TargetPosition` | The current move target, or `null` if not navigating. |
| `Velocity` | Current crowd velocity (mutable — you can set it). |
| `WishVelocity` | The desired velocity the agent wants to move at. |
| `IsNavigating` | True if a valid target is set and the agent is walking or traversing a link. |
| `IsTraversingLink` | True while crossing an off-mesh link. |
| `CurrentLinkTraversal` | Details of the link being traversed (positions, link component). |

## Methods

- `MoveTo( Vector3 target )` — request navigation to a world point.
- `SetAgentPosition( Vector3 position )` — teleport the agent without re-pathing through the world.
- `SetPath( NavMeshPath path )` — assign a precalculated path. Path must start within ~`2 * Radius` of the agent.
- `GetPath()` — returns the agent's current `NavMeshPath`. Not cheap.
- `Stop()` — clear the move target.
- `CompleteLinkTraversal()` — call after a manual link traversal when `AutoTraverseLinks` is false.
- `GetLookAhead( float distance )` — point on the agent's path `distance` units ahead.

## Events

- `LinkEnter` — invoked when the agent enters an off-mesh link.
- `LinkExit` — invoked when the agent leaves an off-mesh link.

:::warning "Agent doesn't move"
The scene needs a generated navmesh. Without `Scene.NavMesh.crowd`, the agent does nothing. Also check that the target you pass to `MoveTo` is actually on a polygon — `MoveTo` silently returns if the nearest poly query fails (search box is `256x256x256` around the target).
:::

:::warning "Agent jitters or fights my controller"
If you're driving the GameObject yourself (e.g., with a `CharacterController`), set `UpdatePosition = false`. Otherwise the agent fights you for the transform.
:::

## Related Pages

- [Character Controller](charactercontroller.md)
- [Player Controller](player-controller.md)
