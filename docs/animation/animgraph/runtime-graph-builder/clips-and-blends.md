---
title: "Clips and Blends"
created: 2026-11-04
updated: 2026-11-05
---

# Clips and Blends

These methods produce poses. Start with clips, combine them, and pass the final result to `builder.Create`.

The fragments below assume `builder` is your `Graph.Builder`, its `Skeleton` is set, and all control parameters have already been declared. Clip variables refer to valid `.anim` resources using that skeleton.

## Play a clip

`Clip` accepts an asset path or a loaded `Sandbox.Animation.Clip`:

```csharp
var wavePose = builder.Clip( waveClip, new GraphBuilder.ClipSettings
{
    Looping = false,
    SampleRootMotion = false
} );
```

Non-looping means **stop advancing at the end**, not remove the pose, leave a state, clear a bool, or disconnect an external graph. Give actions an explicit [completion rule](/animation/animgraph/runtime-graph-builder/state-machines.md).

| `ClipSettings` | Purpose |
|---|---|
| `Looping` | Repeat the clip; defaults to `true` |
| `Speed` | Constant playback multiplier; defaults to `1` |
| `PlayInReverse` | Optional Bool node controlling reverse playback; ignored during synchronized updates |
| `ResetTime` | Optional Bool node requesting a time reset; do not leave it true unintentionally |
| `SyncEventOffset` | Offset in the clip's synchronization events |
| `SampleRootMotion` | Include the clip's root-motion contribution; defaults to `true` |
| `GraphEvents` | Names of generic graph events emitted by the active clip |

For a changing playback multiplier, wrap the clip in `SpeedScale` rather than rebuilding `ClipSettings`.

## Reverse a clip

Declare a Bool before adding any other nodes, then connect it through `ClipSettings`. This fragment assumes a fresh builder with its skeleton already selected and a compatible `waveClip`:

```csharp
var reverse = builder.BoolParameter( "reverse" );
var pose = builder.Clip( waveClip, new GraphBuilder.ClipSettings
{
    Looping = false,
    PlayInReverse = reverse,
    SampleRootMotion = false
} );
```

Create the graph from `pose`, then supply `"reverse"` through `Renderer.Animation.Parameters.Set`. For a graph that always plays backward, use `builder.Constant( true )` as `PlayInReverse` instead of declaring a control.

**This requires unsynchronized playback.** `Blend` itself, `BlendSpace`, `BlendSpace2D`, and `VelocityBlend` synchronize their child clips. So do layers with `Synchronized = true` and synchronized transitions while active. An ordinary clip ignores its reverse input when that parent timing reaches it.

Putting a blend inside an unsynchronized layer does not remove the blend's own synchronization. Use an independent action branch, a deliberately time-controlled clip, or a separately baked reversed asset. See [Reversing Clips](/animation/animgraph/reversing-clips.md) for editor setup and the full restrictions.

## Sample a time instead of playing

| Method | Use it for |
|---|---|
| `TimeControlledClip( clip, time, settings )` | A clip driven by an external normalized time value, rather than its own playback clock |
| `Pose( clip, time )` | Sampling a pose from a Float node |
| `Pose( clip, new GraphBuilder.PoseSettings { ... } )` | A fixed sample, frame input, or a remapped pose-sheet input |
| `ReferencePose()` | The skeleton's authored reference pose |
| `ZeroPose()` | A zero pose, not the skeleton's authored reference pose |

The clip-taking methods accept either paths or loaded clips.

`TimeControlledClip` takes time in the range `0` to `1` and ignores synchronization. Its `TimeControlledClipSettings` provides `PlayInReverse`, `SampleRootMotion` (default `true`), and `GraphEvents`.

`PoseSettings` provides an optional Float `Time`, a `FixedTime` fallback (default `0`), `RemapRangeStart`/`RemapRangeEnd` (default `0`/`1`), and `UseFramesAsInput` (default `false`). A pose sample does not play through the clip by itself.

To play backward from a progress parameter that increases from `0` to `1`, drive the source time backward **and** set the direction:

```csharp
var progress = builder.FloatParameter( "progress", 0, 1 );
var time = builder.Subtract( builder.Constant( 1.0f ), progress );
var pose = builder.TimeControlledClip( waveClip, time,
    new GraphBuilder.TimeControlledClipSettings
    {
        PlayInReverse = builder.Constant( true ),
        SampleRootMotion = false
    } );
```

Use a fresh builder for this alternative, with the skeleton selected before building. The input still determines the sampled source position: the reverse bool does not invert that position for you. It makes event and root-motion sampling follow the backward direction. Decreasing time without that flag is interpreted as a forward loop.

For pose-only sampling, use `builder.Pose( waveClip, time )` with the same decreasing time instead. It does not sample events or root motion.

## Blend two poses

```csharp
var result = builder.Blend( idlePose, walkPose, movementWeight );
```

`movementWeight` is a Float node: `0` uses idle, `1` uses walk, and values between them blend both. There is also a constant-float overload.

`Blend` is a convenience two-point blend space. It is not an automatic timed transition: smooth the weight yourself with `Ease` or `Spring`, or use a state machine.

## Blend by movement speed

```csharp
var movement = builder.BlendSpace( speed )
    .Add( 0, builder.Clip( idleClip ) )
    .Add( 100, builder.Clip( walkClip ) )
    .Add( 300, builder.Clip( runClip ) );
```

The builder sorts the positions and blends neighboring entries. Choose distinct positions that represent your clips. `looping` defaults to `true`; set it explicitly when building a non-looping blend.

The lower-level `BlendSpace( parameter, sources, ranges, looping )` overload accepts ranges as `(Input0, Input1, Start, End)`. Indices refer to `sources`. Prefer the fluent version unless you need to supply that structure yourself.

## Blend on two axes

```csharp
var movement = builder.BlendSpace2D( forwardSpeed, sidewaysSpeed )
    .Add( new Vector2( 0, 0 ), builder.Clip( idleClip ) )
    .Add( new Vector2( 100, 0 ), builder.Clip( forwardClip ) )
    .Add( new Vector2( 0, 100 ), builder.Clip( sidewaysClip ) );
```

These are three corners of a blend space, not a complete directional movement set. Add the other directions your character needs.

The fluent builder triangulates the points. It needs at least three distinct, non-collinear positions and supports at most 256 entries. Inputs outside the space project onto its boundary.

For precomputed data, use `BlendSpace2D( x, y, BlendSpace2DSettings )`. Supply matching `Sources` and `Positions`, triangle index triplets in `Indices`, and the boundary loop in `HullIndices`. `Looping` defaults to `true`.

`GraphBuilder.TriangulateBlendSpace( points, out triangles, out hull )` produces those indices and returns `false` if triangulation is impossible. The hull repeats its first index at the end. Handle failure before creating the graph.

## Change playback rate

| Method | Meaning |
|---|---|
| `SpeedScale( child, speed )` | Multiply playback speed; accepts a Float node or constant |
| `DurationScale( child, duration )` | Adjust playback to a desired duration in seconds; accepts a Float node or constant |
| `DurationScale( child, duration, defaultDuration )` | Use the fallback duration if the optional node is omitted |
| `VelocityScale( animationPath, desiredVelocity, looping )` | Scale a clip using its baked average root-motion speed |
| `VelocityScale( clipNode, desiredVelocity, defaultVelocity )` | Clip-node version with a fallback desired velocity |
| `VelocityBlend( speed, clips )` | Blend clips using their baked average root-motion speeds as positions; supply them slowest first |

Use positive durations and sensible playback speeds. Negative Speed Scale inputs become zero, not reverse playback; negative driven durations and desired velocities are not reverse controls either. Use `PlayInReverse` on a suitable clip instead.

Velocity-based nodes need meaningful authored root motion; they cannot infer foot speed from arbitrary in-place animation. A manually positioned blend space is often the clearer starting point.

## Select once on entry

Pose selectors choose when initialized. Changing their inputs while they are already active does **not** continuously choose a different pose.

For a conditional choice, add an explicit final fallback:

```csharp
var stance = builder.Selector()
    .Add( crouchPose, isCrouching )
    .Add( standingPose, builder.Constant( true ) );
```

The first true condition wins. If none is true, there is no selected pose. Put the always-true fallback last.

| Selector | Choice |
|---|---|
| `Selector().Add( pose, condition )` | First true Bool condition |
| `ClipSelector( (clip, condition), ... )` | The same selection, restricted to clip nodes |
| `Variation( seed ).Add( pose, weight )` | Deterministic weighted choice from a Float seed |
| `ClipVariation( seed, (clip, weight), ... )` | Weighted choice restricted to clips |
| `ParameterizedSelector( parameter, options, settings )` | Lower-level weighted selection |
| `IdBasedSelector( id, options, fallback )` | Match an ID against `(Id, Option)` entries |
| `IdBasedClipSelector( id, options, fallback )` | ID selection restricted to clips |
| `TargetSelector( target, clips, settings )` | Choose the clip whose baked endpoint best matches a target |

**Variation does not generate a new random seed.** Selection uses the integer part of the absolute seed modulo the total weight. With weights `2, 1`, seeds `0` and `1` choose the first option and `2` chooses the second. Weights are integers from `0` to `255`; keep at least one positive.

These builder selectors are not the editor's [named graph variations](/animation/animgraph/variations.md). Named variations override authored node data and compile into separately loadable graph resources.

For random variation, use a suitable range such as `builder.RandomFloat( 0, 1000 )`. The default random range of `0` to `1` is unsuitable here: its integer part almost always selects the first weighted entry. A random node samples on initialization, not every frame.

`ParameterizedSelectorSettings` contains `Weights`, `HasWeights`, `IgnoreInvalidOptions`, and `ClipsOnly`. Set `HasWeights` when supplying weights, with one weight per option.

For ID selectors, supply a fallback explicitly. An unmatched ID without a fallback does not automatically use the first option. The settings overload uses `IdBasedSelectorSettings.Fallback`, `IgnoreInvalidOptions`, and `ClipsOnly`; pass an explicit settings object to distinguish it from the fallback-node overload.

`TargetSelectorSettings` provides `PositionWeight` and `OrientationWeight` (both `1`), `WorldSpace` (default `false`), `AlignmentBone`, and `IgnoreInvalidOptions`. This chooses an authored clip; it does not warp the clip to reach the target. Use [TargetWarp](/animation/animgraph/runtime-graph-builder/procedural-animation.md) for that.

## Clip-only inputs

Some nodes need access to a clip's timing and baked root motion, not just a final pose. Warps and velocity-based operations are examples.

Use `Clip` or a clip-specific selector for those inputs. A generic `Selector`, blend, or state machine is not interchangeable with a clip node, even though all of them produce poses.

`Passthrough( child )` passes a pose through unchanged. It can make generated graph structure explicit, but is not needed around ordinary connections.
