---
title: "Values and Parameters"
created: 2026-11-04
updated: 2026-11-05
---

# Values and Parameters

Control parameters are the graph's inputs. Value nodes calculate with those inputs. Neither is a C# variable containing the current evaluated result: each returned `Node` describes a runtime connection.

Declare **every control parameter before adding any other node**, including constants and calculations.

## Control parameters

| Method | Value supplied by gameplay |
|---|---|
| `FloatParameter( name, min, max )` | `float` |
| `BoolParameter( name )` | `bool` |
| `IdParameter( name, options )` | A named choice supplied as a string |
| `VectorParameter( name )` | `Vector3` |
| `TargetParameter( name )` | An animation target, such as a transform or bone |

Names must be nonblank and unique, case-insensitively, across control and virtual parameters.

The convenience helpers start at zero, false, or an empty/unset value. To choose defaults, use `AddControlParameter`:

```csharp
var speed = builder.AddControlParameter(
    "speed",
    GraphBuilder.NodeKind.Float,
    new GraphBuilder.ControlParameterSettings
    {
        Min = 0,
        Max = 300,
        DefaultFloat = 100
    }
);
```

`ControlParameterSettings` provides `DefaultFloat`, `DefaultBool`, `DefaultID`, and `DefaultVector`, plus `Min`, `Max`, `Options`, and `Describe` (default `true`). `AddControlParameter` supports Bool, Float, Id, Vector, and Target, not Pose or BoneMask.

**Ranges and ID options are descriptions, not runtime restrictions.** Use `Clamp` when a value must stay in bounds. `Describe = false` hides the parameter from descriptions/tools without preventing runtime writes.

Once the graph is assigned:

```csharp
Renderer.Animation.Parameters.Set( "speed", 150.0f );
```

Use [Parameters and Code](/animation/animgraph/parameters-and-code.md) for runtime writes, saved inspector values, and PlayerController inputs. A graph's parameter descriptions are not necessarily a list of every settable input.

## Smooth a speed

After declaring `speed`, build a smoothed value and feed it to a blend:

```csharp
var smoothSpeed = builder.Spring( speed, new GraphBuilder.SpringSettings
{
    Hertz = 4,
    DampingRatio = 1
} );

var movement = builder.BlendSpace( smoothSpeed )
    .Add( 0, builder.Clip( idleClip ) )
    .Add( 100, builder.Clip( walkClip ) );
```

`Ease` approaches changes over a chosen time; `Spring` follows them with a damped spring. Use one deliberately rather than stacking smoothing everywhere.

| Settings | Defaults |
|---|---|
| `EaseSettings` | `Time = 0.2`, `Easing = Linear`, `UseStartValue = false`, `StartValue = 0` |
| `SpringSettings` | `Hertz = 4`, `DampingRatio = 0.7`, `UseStartValue = false`, `StartValue = 0` |

Set `UseStartValue` when the calculation should begin from `StartValue` instead of its initial input. A spring can overshoot; clamp its result if the consumer requires strict bounds.

`GraphBuilder.Easing` includes `Linear`, `None`, and `In`, `Out`, and `InOut` variants of `Quad`, `Cubic`, `Quart`, `Quint`, `Sine`, `Expo`, and `Circ`.

## Constants and logic

| Method | Result |
|---|---|
| `Constant( float )`, `Constant( bool )`, `Constant( Vector3 )` | A value that does not change |
| `ConstantId( string )` | A fixed ID |
| `Not( value )` | Inverted Bool |
| `And( values )`, `Or( values )` | Combined Bool conditions |
| `Compare( input, comparison, comparand, epsilon )` | Float comparison, against another node or a constant |
| `InRange( input, min, max, inclusive )` | Whether a float is in the range; inclusive by default |

`FloatComparison` provides `GreaterThanEqual`, `LessThanEqual`, `GreaterThan`, `LessThan`, and `NearEqual`. Use `epsilon` for the tolerance of `NearEqual`.

```csharp
var moving = builder.Compare(
    speed, GraphBuilder.FloatComparison.GreaterThan, 5.0f
);
var canMove = builder.And( grounded, moving );
```

These are graph conditions, not C# `bool` expressions.

## Float calculations

| Method | Purpose |
|---|---|
| `Add`, `Subtract`, `Multiply`, `Divide`, `Modulo` | Two-input arithmetic; the second input can be a node or float |
| `Abs`, `Negate`, `Floor`, `Ceiling`, `Fraction` | Single-input arithmetic |
| `FloatMath( operation, a, settings )` | General arithmetic operation |
| `Clamp( input, min, max )` | Limit a value |
| `Remap( input, inputRange, outputRange )` | Map between `Vector2` ranges, clamped to the output range |
| `Switch( condition, whenTrue, whenFalse )` | Choose two floats, supplied as nodes or constants |
| `FloatSelector( cases, defaultValue, easeTime, easing )` | First true `(Condition, Value)` case, easing between choices |
| `Curve( input, points )` | Remap through a cubic Hermite curve |
| `AngleMath( input, operation )` | Normalize or flip a degree-valued angle |
| `RandomFloat( min, max )` | Sample on initialization and hold until shutdown |
| `Ease`, `Spring` | Smooth a changing float |

`Switch` is a **float** switch, not a generic switch for poses or targets. Unlike pose selectors, float selection responds to changes while evaluated. `FloatSelector` defaults to value `0`, ease time `0.2` seconds, and linear easing.

`FloatMathOperation` includes the arithmetic above plus `IntegerPart`, `FractionalPart`, and `InverseFractionalPart`. `FloatMathSettings.B` supplies the second node, otherwise `ValueB` is used; `AbsoluteResult` and `NegateResult` modify the result. Avoid zero divisors.

`Curve` takes `(Parameter, Value, InTangent, OutTangent)` points, sorts them, and clamps outside the endpoint range. Its two-value `(Parameter, Value)` overload uses tangents of `1`; it is not an arbitrary piecewise-linear mapping. Use distinct parameter positions.

`AngleOperation` provides `ClampTo180`, `ClampTo360`, `FlipHemisphere`, and `FlipHemisphereNegate`. The clamp operations wrap angles into their named ranges.

`RandomFloat` defaults to `0` to `1`. Bounds must be finite and ordered. For a random clip choice, read the [variation seed rules](/animation/animgraph/runtime-graph-builder/clips-and-blends.md).

## IDs

IDs are named choices, not integer animation indices.

| Method | Purpose |
|---|---|
| `IdCompare( id, matchIds, mustMatch )` | Match one string or a list; `mustMatch = false` inverts the result |
| `IdToFloat( id, mapping, defaultValue )` | Map `(Id, Value)` entries to a float |
| `IdSwitch( condition, whenTrue, whenFalse )` | Choose between ID nodes or strings |
| `IdSwitch( condition, settings )` | Mix node and constant choices |
| `IdSelect( cases, defaultValue )` | First true `(Condition, Value)` string choice |

`IdSwitchSettings` provides `WhenTrue`/`WhenFalse` nodes and `TrueValue`/`FalseValue` string fallbacks. ID value selection is not the same as an entry-only ID-based **pose** selector.

## Vectors

`VectorInfo( vector, component )` extracts a float: `X`, `Y`, `Z`, `Length`, `AngleHorizontal`, or `AngleVertical`. Angles are in degrees.

`VectorCreate( x, y, z, baseVector )` creates a vector from optional Float nodes. Omitted components come from `baseVector`, or zero if no base is supplied. Provide at least one input. `VectorNegate( vector )` reverses the vector.

For horizontal speed, remove the vertical component before measuring length:

```csharp
var horizontalVelocity = builder.VectorCreate(
    z: builder.Constant( 0.0f ),
    baseVector: velocity
);
var speed = builder.VectorInfo(
    horizontalVelocity, GraphBuilder.VectorComponent.Length
);
```

Here `velocity` is an already-declared Vector control parameter; `speed` is a calculation, not another control parameter.

## Targets

A Target can carry position and rotation, refer to a bone, or be unset. It is different from a Vector containing only a point or direction.

| Method | Purpose |
|---|---|
| `ConstantTarget( position, rotation )` or `ConstantTarget( transform )` | Fixed, set target |
| `BoneTarget( boneName )` | Target referencing a character bone |
| `UnsetTarget()` | Explicit no-target value |
| `NullTarget()` | A default unset target node |
| `IsTargetSet( target )` | Bool indicating whether a target is set |
| `TargetInfo( target, info, worldSpace )` | A float derived from the target |
| `TargetPoint( target, worldSpace )` | Extract the target's position |
| `TargetOffset( target, translationOffset, rotationOffset, boneSpace, translation )` | Offset a target, optionally overriding translation with a Vector node |

`TargetInfoType` includes `AngleHorizontal`, `AngleVertical`, `Distance`, `DistanceHorizontalOnly`, `DistanceVerticalOnly`, and `DeltaOrientationX`/`Y`/`Z`.

`TargetInfo` and `TargetPoint` default to world-space targets. `TargetOffset` defaults to applying offsets in the target's local/bone space. Match the coordinate space expected by [IK and warp nodes](/animation/animgraph/runtime-graph-builder/procedural-animation.md).

## Virtual parameters and cached values

Use a virtual parameter to name a calculation:

```csharp
var smoothSpeed = builder.Spring( speed );
builder.BindVirtualParameter( "smooth_speed", smoothSpeed );
```

`BindVirtualParameter` names the existing value node and marks it persistent for the graph's lifetime. This is useful for a spring that should retain its initialized state across branch changes.

The typed helpers `VirtualBoolParameter`, `VirtualFloatParameter`, `VirtualIdParameter`, `VirtualVectorParameter`, `VirtualTargetParameter`, and `VirtualBoneMaskParameter` instead return a named wrapper around their value. These wrappers follow their referencing branches' lifetime; they do not automatically mark the calculation persistent.

Virtual parameters are calculated inside the graph, not written with `Animation.Parameters.Set`. Compatible named values can be reflected into [internal child graphs](/animation/animgraph/internal-graphs.md#supply-a-calculated-parent-value).

`Cached( source, cacheOnInit: true )` latches a Bool, Float, ID, Vector, or Target on initialization. With `cacheOnInit: false`, it caches on branch exit instead. It does not support poses or masks.

Initialization lifetime matters: a branch-local random or cached value can reset on re-entry, while a graph-lifetime calculation stays initialized. Persistent does not mean gameplay parameters are saved to disk or restored after the renderer's controller is recreated.
