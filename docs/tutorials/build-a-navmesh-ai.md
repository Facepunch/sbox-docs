---
title: "Build a NavMesh AI"
icon: "🤖"
sources:
  - engine/Sandbox.Engine/Scene/Components/Navigation/NavMeshAgent.cs
  - engine/Sandbox.Engine/Game/Navigation/NavMesh/NavMesh.cs
  - engine/Sandbox.Engine/Game/Navigation/NavMesh/NavMesh.Query.cs
updated: 2026-04-25
created: 2026-04-27
---

# Build a NavMesh AI

A goal-driven enemy that wanders the level when nothing's interesting and chases the player when they get close. We'll bake a navmesh, drop in a `NavMeshAgent`, write a small Idle/Patrol/Chase state machine, and visualize what state it's in. The same shape scales to anything from monsters to NPCs running errands.

## What you'll build

- A baked NavMesh covering the level's walkable surfaces.
- An `EnemyAI` component using `NavMeshAgent.MoveTo` and `Stop`.
- A finite state machine: `Idle` (pause and look around), `Patrol` (wander to a random navmesh point), `Chase` (head for the player).
- A simple debug indicator above the agent showing its current state.

## Prerequisites

- A scene with a floor and at least a couple of walls — geometry the navmesh has to route around to look interesting.
- A Player GameObject you can move around. The [Third-Person Controller tutorial](build-a-third-person-controller.md) covers building one.
- Familiarity with components ([Your First Component](your-first-component.md)).

---

## Step 1: Bake the NavMesh

The navmesh is a procedural mesh of walkable polygons generated from your level's collision geometry.

1. Open your scene.
2. In the Editor's main toolbar, find the **Enable NavMesh** toggle (the icon is a mesh grid). Click it. This enables NavMesh on the scene's settings.
3. Click **View NavMesh** next to it to see the generated mesh as a translucent overlay. Walkable polygons appear blue/green; un-walkable areas (slopes too steep, gaps, ceilings) are skipped.

The NavMesh component on the scene exposes parameters like agent radius, agent height, max slope, and step height. Defaults work for humanoid characters. For a project-wide configuration, check the **Scene -> Navigation** section in the Inspector when nothing is selected.

> **If the navmesh is empty**, your geometry doesn't have collision. The navmesh generator only sees `Collider` components. Add `BoxCollider`s to your floors and walls if they don't already have them. (`Cube` GameObjects from the Scene Tree create-menu come with one already.)

> **If only part of your floor is covered**, the missing chunk is probably outside the **Bounds** of the NavMesh. Check the scene's NavMesh settings — there's a bounding box you can expand.

### Checkpoint

Press **View NavMesh** and confirm a continuous walkable region across your floor. Walls show up as gaps. If you walk a player around in Play mode, the navmesh covers everywhere you can stand.

---

## Step 2: Build the AI character

1. Create an empty GameObject named `Enemy`. Position it on the navmesh — drop it at `(200, 200, 50)` or somewhere visible.
2. Add a `ModelRenderer`. For a quick blockout use `models/dev/box.vmdl`. Scale it to roughly humanoid: `WorldScale = (0.5, 0.5, 1.5)`.
3. Add a `NavMeshAgent`.

Look at the `NavMeshAgent` properties in the Inspector. The **Physical Properties** group has `Height` (default 64) and `Radius` (default 16) — these should match the agent's body, since the navmesh uses them to validate paths. The **Movement** group has:

- `MaxSpeed` (default 120) — top movement speed in units/sec.
- `Acceleration` (default 120) — how fast it can change direction. Equal to `MaxSpeed` is "snappy", smaller is "drifty".
- `Update GameObject Position` — when on, the agent moves the GameObject's `WorldPosition` for you. Leave it on.
- `Update GameObject Rotation` — off by default. The agent will face its direction of travel; turn it off if you have your own animation that handles facing.

> **If the agent doesn't move at runtime**, you almost certainly placed it off the navmesh. The `NavMeshAgent` only moves when its starting point projects onto a valid polygon. Drop the agent in editor right onto a green/blue navmesh region.

### Checkpoint

Press Play. The agent stands still — we haven't told it to do anything yet. Stop the game.

---

## Step 3: A trivial first move

Before the state machine, prove the agent can move. Create `EnemyAI.cs`:

```csharp
using Sandbox;

public sealed class EnemyAI : Component
{
    [RequireComponent] public NavMeshAgent Agent { get; set; }

    [Property] public Vector3 InitialDestination { get; set; } = new Vector3( 0, 0, 0 );

    protected override void OnStart()
    {
        Agent.MoveTo( InitialDestination );
    }
}
```

Add this to `Enemy`. Press Play. The agent should walk to the origin, recompute around obstacles if needed, and stop when it arrives.

> **If `Agent` is null at runtime**, `[RequireComponent]` populates it on the *editor* path when adding the component. If you wrote the script *before* attaching, the agent reference may be empty. Remove the `EnemyAI` component and re-add it; the editor will wire it up.

> **If the agent walks and then immediately stops**, watch `Agent.IsNavigating` in the Inspector while running — it goes false the moment the path is complete or invalidates. If it never went true, the destination didn't project onto the navmesh.

### Checkpoint

You see the box visibly travel from its start position to (0,0,0), avoiding any walls. If the path is straight even through a wall, your geometry isn't blocking the navmesh — go back to step 1.

---

## Step 4: A state machine

Replace the contents of `EnemyAI.cs`:

```csharp
using Sandbox;

public enum AiState
{
    Idle,
    Patrol,
    Chase,
}

public sealed class EnemyAI : Component
{
    [RequireComponent] public NavMeshAgent Agent { get; set; }

    [Property] public GameObject PlayerTarget { get; set; }

    [Property] public float ChaseDistance { get; set; } = 500f;
    [Property] public float LoseDistance { get; set; } = 800f;
    [Property] public float IdleDuration { get; set; } = 2f;
    [Property] public float PatrolRange { get; set; } = 1000f;

    [Property] public AiState State { get; private set; } = AiState.Idle;

    private TimeSince _timeInState;

    protected override void OnUpdate()
    {
        if ( Agent is null || Scene.NavMesh is null ) return;

        switch ( State )
        {
            case AiState.Idle: TickIdle(); break;
            case AiState.Patrol: TickPatrol(); break;
            case AiState.Chase: TickChase(); break;
        }
    }

    void TransitionTo( AiState next )
    {
        if ( State == next ) return;
        State = next;
        _timeInState = 0;
        Agent.Stop();
    }

    void TickIdle()
    {
        // If the player is close enough, escalate to chase.
        if ( PlayerInRange( ChaseDistance ) )
        {
            TransitionTo( AiState.Chase );
            return;
        }

        // Otherwise, after a beat, go patrolling.
        if ( _timeInState >= IdleDuration )
        {
            TransitionTo( AiState.Patrol );
        }
    }

    void TickPatrol()
    {
        // Player got close - drop the patrol and chase.
        if ( PlayerInRange( ChaseDistance ) )
        {
            TransitionTo( AiState.Chase );
            return;
        }

        // Pick a destination if we don't have one.
        if ( !Agent.IsNavigating )
        {
            var origin = WorldPosition;
            var destination = Scene.NavMesh.GetRandomPoint( origin, PatrolRange );
            if ( destination.HasValue )
            {
                Agent.MoveTo( destination.Value );
            }
            else
            {
                // No valid spot found - back to idle.
                TransitionTo( AiState.Idle );
                return;
            }
        }

        // We've reached the patrol point.
        if ( _timeInState > 0.5f && !Agent.IsNavigating )
        {
            TransitionTo( AiState.Idle );
        }
    }

    void TickChase()
    {
        if ( PlayerTarget is null )
        {
            TransitionTo( AiState.Idle );
            return;
        }

        if ( !PlayerInRange( LoseDistance ) )
        {
            TransitionTo( AiState.Idle );
            return;
        }

        // Re-target every frame; the player moves.
        Agent.MoveTo( PlayerTarget.WorldPosition );
    }

    bool PlayerInRange( float range )
    {
        if ( PlayerTarget is null ) return false;
        return Vector3.DistanceBetween( WorldPosition, PlayerTarget.WorldPosition ) <= range;
    }
}
```

> **Pause and predict:** Look at the two distance thresholds — `ChaseDistance = 500` and `LoseDistance = 800`. What would go wrong if we only had one threshold (say, `ChaseDistance = 500`) and used it for *both* "start chasing" and "stop chasing"? Picture the player walking right at the boundary.
>
> _Continue reading once you've made a guess._

A few details worth calling out:

- **Two distance thresholds.** `ChaseDistance` (closer) starts the chase; `LoseDistance` (farther) gives up. Without the gap (a hysteresis loop), the AI flickers in and out of chase exactly at the boundary.
- **`Agent.Stop()` on transition.** Resets the agent's destination so the new state starts clean. Without it, `TickPatrol` would inherit a stale chase target on its first frame.
- **`Scene.NavMesh.GetRandomPoint( origin, radius )`** — this overload picks a random navmesh point within `radius` of `origin`. The parameterless `GetRandomPoint()` picks anywhere on the entire navmesh, which can send your enemy across the level.
- **`Agent.IsNavigating`** — true while the agent has a target it hasn't reached. We use it as a "still walking?" check.

> **If `Scene.NavMesh.GetRandomPoint(...)` returns null**, the navmesh hasn't finished baking, or the radius is small enough that no polygon falls inside it. Fall back to idle, as the example does.

### Hook it up

1. Save the file. Wait for compile.
2. Select your `Enemy` GameObject. The `EnemyAI` properties appear on the existing component.
3. Drag the `Player` GameObject from the Scene Tree into the `PlayerTarget` slot.

### Checkpoint

Press Play and stand far from the enemy. It picks a destination on the navmesh, walks there, idles for two seconds, picks another, and continues. The `State` field in the Inspector ticks `Idle -> Patrol -> Idle -> Patrol`.

Now walk close (within 500 units). The state flips to `Chase` and the enemy comes after you. Run away past 800 units; it gives up and idles.

---

## Step 5: Visualize the state

> **Pause and predict:** `TransitionTo` calls `Agent.Stop()` on every state change. If we deleted that one line, what specific bug would show up — and in which transition would you notice it first?
>
> _Continue reading once you've made a guess._

State machines are easier to debug when you can see the current state at a glance. Add this to `EnemyAI`:

```csharp
protected override void OnUpdate()
{
    // ... existing switch ...

    DrawDebugIndicator();
}

void DrawDebugIndicator()
{
    var color = State switch
    {
        AiState.Idle => Color.Gray,
        AiState.Patrol => Color.Yellow,
        AiState.Chase => Color.Red,
        _ => Color.White,
    };

    var pos = WorldPosition + Vector3.Up * 100f;
    DebugOverlay.Sphere( new Sphere( pos, 8f ), color, duration: 0 );
    DebugOverlay.Text( pos + Vector3.Up * 12f, State.ToString(), color: color, size: 18 );
}
```

`DebugOverlay` is a per-component overlay system that renders directly to the editor view. `duration: 0` means "this frame only", so we redraw every update.

> **If the indicator doesn't appear**, your scene viewport is probably running with debug overlays disabled. Open the **viewport options** (the gear icon in the upper right of the 3D view) and ensure **Debug Overlay** rendering is on.

### Checkpoint

While the enemy patrols, a yellow sphere bobs above its head. Walk into chase range — the sphere flips red. Stop the game and the overlay clears.

---

:::danger Agent stutters / oscillates
Two causes. (1) You're calling `MoveTo` every frame with the same target — actually, `MoveTo` is hardened against this internally (it re-checks before re-pathing), so the more likely issue is rapid state transitions. Add hysteresis: a separate `LoseDistance` larger than `ChaseDistance`, as the tutorial does. (2) The agent and player are the same height and overlapping their `Radius` — the agent thinks it's "stuck" against the player's body. Increase the player's height or use the agent's `Separation` slider.
:::

:::warning Agent sinks into the floor
The navmesh sits *just above* the collision geometry. If your `NavMeshAgent.Height` doesn't match the actual model height, the agent's GameObject pivot drifts. Either match `Height` to your model, or set `Update GameObject Position` off and reposition the visual yourself.
:::

:::warning No path found, agent stays still
The starting position isn't on the navmesh, or the destination isn't. The `MoveTo` method silently fails when the destination doesn't project onto a valid polygon — it doesn't log. Add a `Log.Info` next to your `MoveTo` calls during debugging.
:::

## Try extending it

- **Hearing.** Add a `[Property] float HearingRange = 1500f;` and a `bool HeardSomething` flag the player sets when running. If the AI hears the player but isn't in chase range, transition to a `Search` state that walks to the last known position.
- **Patrol points.** Replace random patrols with an array `[Property] List<GameObject> Waypoints { get; set; }`. The AI walks to each in order. (Use `Scene.NavMesh.GetClosestPoint( waypoint.WorldPosition )` to snap to the navmesh.)
- **Multiple targets.** Track all components implementing some `IThreat` interface and chase the closest one.
- **Animation.** The agent exposes `Velocity` and `WishVelocity`. Feed `Velocity.Length / MaxSpeed` into a `SkinnedModelRenderer`'s parameter to drive walk/run blends.

## Try it yourself

Now add a **fourth state: `Flee`**. The behavior: when the AI's *own* health drops below a threshold, it stops chasing and runs to a navmesh point *away* from the player until it heals or the player is far enough away.

This builds entirely on what you've used:

- Add `Flee` to the `AiState` enum.
- Give `EnemyAI` a `[Property] public PlayerHealth SelfHealth { get; set; }` (or just a `float CurrentHealth` if you didn't do the controller tutorial). Drag the AI's own health component onto it in the Inspector — or use `Components.Get<PlayerHealth>()` in `OnStart`.
- In every state's tick, check the health threshold first. If low, `TransitionTo( AiState.Flee )`.
- Implement `TickFlee`. The trick: pick a destination *away* from the player. `Scene.NavMesh.GetRandomPoint( origin, radius )` already exists — pass an `origin` that's offset away from the player, not the AI's own position.
- Add `Flee` to the `DrawDebugIndicator` switch with a distinct color.

Try it before reading on. Get stuck? Here's a hint:

<details>
<summary>Hint 1</summary>

For "away from the player", compute a direction:

```csharp
var awayDir = (WorldPosition - PlayerTarget.WorldPosition).Normal;
var fleeOrigin = WorldPosition + awayDir * 600f;
var dest = Scene.NavMesh.GetRandomPoint( fleeOrigin, 300f );
```

That picks a random point in a 300-unit blob centered 600 units further away from the player. The blob shape is intentional — pure straight-away flees feel mechanical.

</details>

<details>
<summary>Hint 2 (the answer)</summary>

```csharp
public enum AiState
{
    Idle,
    Patrol,
    Chase,
    Flee,
}

[Property] public float FleeBelowHealth { get; set; } = 25f;
[Property] public float CurrentHealth { get; set; } = 100f;

protected override void OnUpdate()
{
    if ( Agent is null || Scene.NavMesh is null ) return;

    // Universal escape: low health overrides any current state except itself.
    if ( State != AiState.Flee && CurrentHealth < FleeBelowHealth )
    {
        TransitionTo( AiState.Flee );
    }

    switch ( State )
    {
        case AiState.Idle: TickIdle(); break;
        case AiState.Patrol: TickPatrol(); break;
        case AiState.Chase: TickChase(); break;
        case AiState.Flee: TickFlee(); break;
    }

    DrawDebugIndicator();
}

void TickFlee()
{
    // Recovered or player gone? Drop back to idle.
    if ( CurrentHealth >= FleeBelowHealth || PlayerTarget is null )
    {
        TransitionTo( AiState.Idle );
        return;
    }

    if ( !Agent.IsNavigating )
    {
        var awayDir = (WorldPosition - PlayerTarget.WorldPosition).Normal;
        var fleeOrigin = WorldPosition + awayDir * 600f;
        var dest = Scene.NavMesh.GetRandomPoint( fleeOrigin, 300f );

        if ( dest.HasValue )
            Agent.MoveTo( dest.Value );
        else
            TransitionTo( AiState.Idle );
    }
}

void DrawDebugIndicator()
{
    var color = State switch
    {
        AiState.Idle => Color.Gray,
        AiState.Patrol => Color.Yellow,
        AiState.Chase => Color.Red,
        AiState.Flee => Color.Cyan,
        _ => Color.White,
    };
    // ... rest unchanged
}
```

Two design notes:

- The "low health" check sits *outside* the switch, so it can preempt any state. That's a common FSM pattern when one condition trumps everything.
- We use a hysteresis-friendly threshold here too: if you wanted to avoid flicker between `Flee` and `Idle` right at the boundary, give `Flee` a higher recovery threshold (e.g., flee below 25, recover above 40).

</details>

## Next steps

- See [NavMesh Areas](../systems/navigation/navmesh-areas/index.md) to mark some surfaces as costlier or forbidden — a common way to keep AI away from danger.
- For more complex pathing (waypoints with off-mesh links, jumps), look at [NavMesh Links](../systems/navigation/navmesh-links.md).
- Combine this with [Networking Basics](networking-basics.md) to make a server-authoritative AI: the agent only ticks on the host, and other clients see its position via networked transform.
