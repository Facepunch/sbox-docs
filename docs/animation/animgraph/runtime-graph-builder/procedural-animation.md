---
title: "Procedural Animation"
created: 2026-11-04
updated: 2026-11-04
---

# Procedural Animation

Procedural nodes adjust an existing animation pose or its root motion. Start with a working clip or blend, then add one adjustment at a time.

All examples below use one builder with its controls already declared. Bone names, masks, targets, and clips must match your character.

## Reach a target with two-bone IK

Declare a Target parameter such as `hand_target` before adding pose nodes. Then:

```csharp
var result = builder.TwoBoneIK(
    movement,
    "hand_R",
    handTarget,
    new GraphBuilder.TwoBoneIKSettings
    {
        WorldSpace = true,
        BlendTime = 0.2f
    }
);
```

`"hand_R"` must be an actual effector bone in your skeleton. The solver rotates its two parents, such as elbow and shoulder, to reach the target and aligns the effector to the target rotation.

The target is a **Target node**, not a Vector. Supply a target transform through the AG2 parameter API, not legacy `SetIk`.

| `TwoBoneIKSettings` | Purpose |
|---|---|
| `Enabled` | Optional Bool switch |
| `Weight` | Optional Float influence |
| `BlendTime` | Blend-in/out time; default `0.2` seconds |
| `WorldSpace` | Target coordinate space; default `true` |
| `BlendMode` | `Effector` (default) or `Pose` |
| `ChainRotationWeight` | Contribution of chain rotation adjustment |

`IKBlendMode.Effector` blends the effector target before solving. `Pose` blends the pre-IK and solved poses and is more expensive.

## Foot IK

`FootIK( child, leftFootBone, rightFootBone, settings )` solves the two legs with independent targets:

```csharp
var result = builder.FootIK( movement, "foot_L", "foot_R",
    new GraphBuilder.FootIKSettings
    {
        LeftTarget = leftTarget,
        RightTarget = rightTarget,
        WorldSpace = true,
        BlendTime = 0.15f
    } );
```

Use your skeleton's foot names. Supply at least one of `LeftTarget` and `RightTarget`; a single supplied target solves only that foot.

`FootIKSettings` also provides `Enabled` and `BlendMode`. Its defaults differ from two-bone IK: `WorldSpace = false` and `BlendTime = 0`. Set them explicitly to avoid mixing world targets with character-space interpretation.

This node does not perform ground traces. Gameplay must calculate suitable targets and decide when feet should be planted.

## Aim and look

`AimAt( child, worldTarget )` consumes a **Vector containing a world-space target position**. It is not the same input type as two-bone IK's Target.

`LookAt( child, pitch, yaw, settings )` distributes degree-valued angles across a named bone chain:

```csharp
var result = builder.LookAt( movement, pitch, yaw,
    new GraphBuilder.LookAtSettings
    {
        ReferenceBone = "pelvis",
        Bones = new[]
        {
            new GraphBuilder.LookAtBone( "head", 1.0f ),
            new GraphBuilder.LookAtBone( "neck", 0.5f )
        },
        YawLimit = 70,
        PitchLimit = 45
    } );
```

The names illustrate the structure; replace them with actual bones in your skeleton. List chain bones **end-to-root**, with unique names and non-increasing weights between `0` and `1`. The builder accepts 1 to 255 chain bones.

`LookAtSettings` requires `ReferenceBone` and `Bones`. It also provides `ReferenceRotation` (normalized, default identity), optional Float `Weight`, `YawLimit`, and `PitchLimit` (both default to `140` degrees, valid range `0` to `180`).

## Warp authored root motion

Warping adjusts a clip's authored movement. It needs a **clip node**, or a clip-specific selector, rather than an arbitrary blended pose.

| Method | Purpose |
|---|---|
| `OrientationWarp( clip, target, relativeToCharacter, samplingMode, alignmentMode )` | Redirect movement/facing using an orientation-warp event |
| `TargetWarp( clip, target, settings )` | Adjust root motion to reach a world-space target transform using target-warp events |

Author the corresponding warp events on the clip first. These nodes are not generic movement controllers and do not invent suitable motion from an unmarked stationary clip.

`OrientationWarp` accepts a character-space Vector direction or a Float angle offset in degrees. `relativeToCharacter` defaults to `true`. `OrientationWarpAlignmentMode` selects `MovementDirection` (default) or `CharacterForward`.

`TargetWarp` takes a Target node. Its settings are:

| `TargetWarpSettings` | Purpose / default |
|---|---|
| `UpdateRule` | `None`, `Recalculate`, `Offset`, or `RecalculateOrOffset`; default `None` |
| `SamplingMode` | How warped root motion is sampled; default `WorldSpace` |
| `AlignAtLastWarpEvent` | Align at the last marked warp event |
| `AlignmentBone` | Optional bone used for alignment |
| `SamplingPositionErrorThreshold` | Position-error tolerance; `0.05` |
| `MaxTangentLength` | Maximum warp-curve tangent length; `1.25` |
| `LerpFallbackDistance` | Distance threshold for the linear fallback; `0.1` |
| `TargetUpdateDistance` | Movement threshold for target updates; `0.1` |
| `TargetUpdateAngleDegrees` | Rotation threshold for target updates; `5` degrees |

`RootMotionSamplingMode.Delta` samples a per-update delta; `WorldSpace` matches the warped world-space path. Both warp methods default to `WorldSpace`.

Changing a target does not imply continuous recalculation: choose a `TargetWarpUpdate` rule when the target is allowed to move during the action.

## Override root motion from gameplay

`RootMotionOverride( child, settings )` replaces movement velocity, facing direction, or both:

```csharp
var result = builder.RootMotionOverride( movement,
    new GraphBuilder.RootMotionOverrideSettings
    {
        MovingVelocity = desiredVelocity,
        FacingDirection = desiredFacing
    } );
```

Both inputs are character-space Vectors. Supply at least one.

`RootMotionOverrideSettings` also provides Float nodes `LinearVelocityLimit` and `AngularVelocityLimit`, constant `MaxLinearVelocity` and `MaxAngularVelocityDegrees` limits (both default `-1`, meaning no constant limit), and `Flags`.

Linear limits are movement units per second. Both the angular-limit node and constant use degrees per second.

`RootMotionOverrideFlags` combines `MoveX`, `MoveY`, `MoveZ`, `FacingPitch`, and `ListenForEvents`; `None` is also available. By default all three movement axes are enabled. `ListenForEvents` makes the override respect authored root-motion override events.

Root-motion generation and moving a GameObject are separate responsibilities. Decide how your movement code consumes the animation's motion; adding a warp or override is not a replacement for collision handling or a character controller.
