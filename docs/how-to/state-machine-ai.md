---
title: "Create a State Machine AI"
icon: "🤖"
sources:
  - engine/Sandbox.Engine/Scene/Components/Navigation/NavMeshAgent.cs
  - engine/Sandbox.Engine/Game/Navigation/NavMesh/NavMesh.cs
created: 2026-04-27
updated: 2026-04-27
---

# Create a State Machine AI

Learn how to build a state machine to make an AI character seamlessly transition between behaviors like patrolling and chasing.

## Quick Working Example

```csharp
using Sandbox;
using System.Collections.Generic;

public sealed class GuardAI : Component
{
	public enum AIState
	{
		Idle,
		Patrol,
		Chase
	}

	[RequireComponent] public NavMeshAgent Agent { get; set; }
	[Property] public GameObject PlayerTarget { get; set; }
	[Property] public List<Vector3> PatrolPoints { get; set; } = new();

	[Property] public float ChaseDistance { get; set; } = 800f;
	[Property] public float AttackDistance { get; set; } = 150f;

	private AIState _currentState = AIState.Idle;
	private int _currentPatrolIndex = 0;
	private TimeSince _timeSinceIdle;

	protected override void OnUpdate()
	{
		if ( PlayerTarget == null || Scene.NavMesh == null ) return;

		// 1. Determine if we need to switch states
		var distanceToPlayer = Vector3.DistanceBetween( GameObject.WorldPosition, PlayerTarget.WorldPosition );

		if ( distanceToPlayer < ChaseDistance )
		{
			SetState( AIState.Chase );
		}
		else if ( _currentState == AIState.Chase )
		{
			SetState( AIState.Idle ); // Lost the player
		}

		// 2. Execute the current state's logic
		switch ( _currentState )
		{
			case AIState.Idle:
				UpdateIdle();
				break;
			case AIState.Patrol:
				UpdatePatrol();
				break;
			case AIState.Chase:
				UpdateChase();
				break;
		}
	}

	private void SetState( AIState newState )
	{
		if ( _currentState == newState ) return;

		// Exit current state logic
		if ( _currentState == AIState.Chase )
		{
			Agent.Stop();
		}

		// Enter new state logic
		_currentState = newState;
		
		if ( _currentState == AIState.Idle )
		{
			Agent.Stop();
			_timeSinceIdle = 0;
		}
	}

	private void UpdateIdle()
	{
		// Wait 3 seconds, then start patrolling
		if ( _timeSinceIdle > 3f )
		{
			SetState( AIState.Patrol );
		}
	}

	private void UpdatePatrol()
	{
		if ( PatrolPoints.Count == 0 ) return;

		// Are we close to our current patrol point?
		var targetPos = PatrolPoints[_currentPatrolIndex];
		if ( Vector3.DistanceBetween( GameObject.WorldPosition, targetPos ) < 50f )
		{
			// Move to the next point
			_currentPatrolIndex = ( _currentPatrolIndex + 1 ) % PatrolPoints.Count;
			SetState( AIState.Idle ); // Pause briefly at each point
			return;
		}

		Agent.MoveTo( targetPos );
	}

	private void UpdateChase()
	{
		var distanceToPlayer = Vector3.DistanceBetween( GameObject.WorldPosition, PlayerTarget.WorldPosition );
		
		if ( distanceToPlayer < AttackDistance )
		{
			Agent.Stop();
			Log.Info( "Attacking Player!" );
			// Perform attack animation or logic here
		}
		else
		{
			// Keep updating path to the moving player
			Agent.MoveTo( PlayerTarget.WorldPosition );
		}
	}
}
```

### Advanced Animation Synchronization

You often want your AI's animations to match its state or movement speed. `NavMeshAgent` exposes `Velocity` and other properties that you can feed directly into your `SkinnedModelRenderer` parameters.

```csharp
[RequireComponent] public SkinnedModelRenderer Renderer { get; set; }

protected override void OnUpdate()
{
	// Send the agent's current speed to the animation graph
	var speed = Agent.Velocity.Length;
	Renderer.Set( "move_speed", speed );

	// Tell the animator if we are falling or jumping (e.g., when traversing links)
	Renderer.Set( "b_grounded", !Agent.IsTraversingLink );
}
```

### NavMesh Avoidance vs Raycasting

`NavMeshAgent` automatically uses crowd avoidance to walk around other agents. However, it does not "see" dynamic rigidbodies unless they have a `NavMeshObstacle` attached. 

If you are building a shooter AI, you might need to combine NavMesh pathfinding with standard physics traces (raycasts) to ensure the AI has line-of-sight before transitioning to an Attack state.

```csharp
private bool CanSeePlayer()
{
	var tr = Scene.Trace.Ray( GameObject.WorldPosition + Vector3.Up * 50f, PlayerTarget.WorldPosition + Vector3.Up * 50f )
		.IgnoreGameObjectHierarchy( GameObject )
		.Run();

	return tr.Hit && tr.GameObject == PlayerTarget;
}
```

## Configuration

When using `NavMeshAgent` inside your states, the following methods and properties are essential:

| Method / Property | Description |
|---|---|
| `Agent.MoveTo( Vector3 pos )` | Calculates a path to the position and begins moving the agent along it. |
| `Agent.Stop()` | Halts the agent's movement immediately and clears its current path. |
| `Agent.Velocity` | The current movement velocity vector of the agent. |
| `Agent.IsTraversingLink` | Returns true if the agent is currently jumping or climbing across a NavMesh Link. |
| `Agent.GetLookAhead( distance )` | Returns a point on the current path a specific distance ahead, useful for aiming or smooth turning. |

## Troubleshooting

:::danger "Agent keeps twitching between states!"
This usually happens when your transition conditions overlap or lack a "deadzone". For example, if you switch to Chase at `Distance < 500` and switch to Idle at `Distance > 500`, a player standing exactly at 500 will cause the AI to rapidly switch back and forth. Fix this by making the escape distance larger than the chase distance (e.g., Chase at `Distance < 500`, Idle at `Distance > 600`).
:::

:::warning "Agent gets stuck on corners or objects"
If the agent is clipping through walls or getting stuck, ensure the `Radius` and `Height` on the `NavMeshAgent` component are large enough to encompass your character model, and that your level's physics colliders accurately match the visuals.
:::

## Related Pages
- [NavMesh Agent](../systems/navigation/navmesh-agent.md)
- [Collisions and Triggers](../systems/physics/collisions-and-triggers.md)
