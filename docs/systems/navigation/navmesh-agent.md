---
title: "NavMesh Agent"
icon: "🛞"
created: 2024-02-02
updated: 2025-11-03
sources:
  - engine/Sandbox.Engine/Scene/Components/Navigation/NavMeshAgent.cs
---

# NavMesh Agent

A `NavMeshAgent` component automatically handles moving a GameObject from position to position on the NavMesh. It features built-in crowd control, meaning agents will try to avoid bumping into each other if possible.

![Crowd of agents moving to the same target. 1698x918](./images/crowd-of-agents-moving-to-the-same-target.mp4)

## Quick Working Example

You can direct a `NavMeshAgent` via your own custom AI script by calling `MoveTo()`. 

```csharp
public class SimpleAIController : Component
{
    [RequireComponent] NavMeshAgent Agent { get; set; }
    
    [Property] public GameObject TargetObject { get; set; }

    protected override void OnUpdate()
    {
        if ( TargetObject == null ) return;

        // Sets the target position for the agent. It will calculate a path and try to get there
        // until you tell it to stop.
        Agent.MoveTo( TargetObject.WorldPosition );
    }
}
```

### Stopping Movement

If you need the agent to halt its current pathing immediately, use `Stop()`.

```csharp
// The agent stops pathing and sets its state to walking, waiting for the next command
Agent.Stop();
```

### Manual Position Syncing

Sometimes you want the `NavMeshAgent` to only calculate the path and desired velocity, but you want a custom `CharacterController` to actually apply the movement and handle gravity. You can turn off `UpdatePosition`.

```csharp
public class CustomAgentMovement : Component
{
    [RequireComponent] NavMeshAgent Agent { get; set; }
    [RequireComponent] CharacterController Controller { get; set; }

    protected override void OnStart()
    {
        // Tell the agent not to move the GameObject itself
        Agent.UpdatePosition = false;
        Agent.UpdateRotation = false;
    }

    protected override void OnUpdate()
    {
        // Tell the agent where we want to go
        Agent.MoveTo( new Vector3( 100, 100, 0 ) );

        // Use the agent's desired WishVelocity to drive our CharacterController
        var wishVel = Agent.WishVelocity;
        
        Controller.Accelerate( wishVel );
        Controller.Move();

        // The agent tracks its own simulated position; you must sync it back manually
        // if your CharacterController handled collisions/gravity that the NavMesh agent didn't foresee
        Agent.SetAgentPosition( WorldPosition );
    }
}
```

### Retrieving the Calculated Path

If you need to draw debug lines or visualize the path the agent intends to take, you can retrieve the current path. Note that this is not a free operation, so avoid calling it every frame in hot loops.

```csharp
if ( Agent.IsNavigating )
{
    NavMeshPath currentPath = Agent.GetPath();

    if ( currentPath.Status == NavMeshPathStatus.Complete )
    {
        // Example: Draw the path using Gizmos in Editor
        foreach ( var point in currentPath.Points )
        {
            Gizmo.Draw.SolidSphere( point.Position, 5f );
        }
    }
}
```

## Configuration

| Property | Description |
|:---|:---|
| **Height** | The height of the agent's bounding cylinder. |
| **Radius** | The radius of the agent's bounding cylinder. |
| **MaxSpeed** | The maximum speed the agent can travel. |
| **Acceleration** | How fast the agent can change its velocity. For snappy movement, set this as high or higher than MaxSpeed. |
| **UpdatePosition** | If true, the agent will automatically set the GameObject's `WorldPosition` every frame. |
| **UpdateRotation** | If true, the agent will automatically face the direction it is moving. |
| **Separation** | Controls how strongly agents avoid crowding each other (0 to 1 scale). |
| **Velocity** | The agent's actual current velocity. |
| **WishVelocity** | (Read-Only) The velocity the agent *wants* to move at to follow its path. Useful for driving a custom `CharacterController`. |
| **IsNavigating** | (Read-Only) Returns true if the agent is currently following a path to a target. |

## Troubleshooting

:::danger "My agent isn't moving to the target!"
Verify that you have actually baked a NavMesh in the scene. The agent cannot calculate a path if there is no NavMesh available or if the target position is off the navigable mesh entirely.
:::

:::warning "My agent is floating or sinking into the floor!"
The `NavMeshAgent` uses a simplified representation of the ground geometry. Since the NavMesh lacks precise height details, it's common for agents to appear slightly offset. Adjust the `Height` and `Radius` properties to better match your visual model, or turn off `UpdatePosition` and drive a precise `CharacterController` using the agent's `WishVelocity`.
:::

## Related Pages
* [Navigation Overview](index.md)
* [Character Controller](../../scene/components/reference/charactercontroller.md)