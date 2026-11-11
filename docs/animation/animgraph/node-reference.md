---
title: "Node Reference"
created: 2026-11-04
updated: 2026-11-11
---

# Node Reference

This is a quick reference to the AnimGraph editor's nodes, grouped by what you want to do. Use the node names when searching the right-click menu. Nodes with an assigned clip, parameter, or custom name may show that name on the graph instead.

Not every node is available in every graph. Pose nodes belong in pose graphs; some conditions only make sense inside a state-machine transition. Output and other editor-managed nodes appear when you create their owning graph.

Start with [Your First Graph](/animation/animgraph/getting-started.md) if you are new to AnimGraph. For the C# equivalents, see [Runtime Graph Builder](/animation/animgraph/runtime-graph-builder/index.md); editor names and builder method names are not always identical.

## Animation and playback

| Node | What it does / when to use it |
|---|---|
| Animation Clip | Plays an `.anim`, with looping, speed, reverse, time reset, events, and root-motion options. The usual starting point for an animation. |
| Time Controlled Animation Clip | Samples a clip at a Float Time input instead of advancing its own clock. Ignores parent synchronization; useful for directly controlled playback. |
| Animation Pose | Samples one pose at a fixed or driven time/frame, without playback events or root motion. Useful for pose sheets and manually positioned mechanisms. |
| Reference Pose | Outputs the skeleton's authored reference pose. Useful as an intentional fallback while building a graph. |
| Zero Pose | Outputs a zero pose, not the skeleton's authored reference pose. It is not a substitute for an idle animation. |
| Speed Scale | Multiplies a child pose's playback speed. Negative input becomes zero, not reverse playback. |
| Duration Scale | Adjusts a child's playback speed to a requested duration in seconds, such as matching a reload's gameplay duration. |
| Velocity Based Speed Scale | Adjusts an Animation Clip's playback rate using its baked average root-motion speed and a desired speed. Requires meaningful authored motion. |

A non-looping clip holds its last pose; it does not leave its state or disconnect itself. See [States and Transitions](/animation/animgraph/states-and-transitions.md).

**Play In Reverse is ignored on an ordinary clip while a synchronized parent supplies its time.** See [Reversing Clips](/animation/animgraph/reversing-clips.md) before reversing clips inside blends or layers.

## Blending and layers

| Node | What it does / when to use it |
|---|---|
| Blend 1D | Blends synchronized poses along one Float axis, such as idle, walk, and run by speed. |
| Blend 2D | Blends synchronized poses across a triangulated two-axis space, such as forward and sideways movement. |
| Blend 1D (Velocity) | Blends synchronized clips using their baked average root-motion speeds as axis positions. |
| Layer Blend | Combines a base pose with Local Layer or State Machine Layer inputs, in layer order. |
| Local Layer | Describes an ordinary layer's pose, weight, mask, blend mode, and timing. Connect its Layer output to Layer Blend. |
| State Machine Layer | Describes an action layer driven by a state machine. Keeps that machine evaluating at zero weight so it can leave Off and finish transitions. |
| Scale | Uses mask weights as per-bone scale factors, or a uniform Default Scale without a mask. It does not mean "blend this animation by a mask." |

Use Overlay for ordinary poses and Additive for clips authored as additive deltas. Independent actions usually should not synchronize to locomotion.

See [Blending Movement](/animation/animgraph/blending-movement.md) and [Layers and Masks](/animation/animgraph/layers-and-masks.md) for complete examples.

## Pose and clip selectors

**These selectors choose on initialization, not continuously.** Use a state machine or blend when an input change should change the active animation.

| Node | What it does / when to use it |
|---|---|
| Selector | Selects the first pose whose condition is true. Add an explicit final true condition if you need a fallback. |
| Clip Selector | The conditional selector restricted to clips, preserving clip access for nodes such as warps. |
| Parameterized Selector | Makes a deterministic weighted pose choice from a Float input. Useful for variations selected on branch entry. |
| Parameterized Clip Selector | The weighted selector restricted to clips, preserving clip-specific access. |
| ID Based Selector | Selects a pose by matching an ID, such as a named stance. Uses its connected Fallback when no option matches. |
| ID Based Clip Selector | The ID-based selector restricted to clips. |
| Target Selector | Chooses the clip whose root motion best matches a target position and orientation. Selection alone does not warp it to reach the target. |

A Parameterized Selector does not generate randomness by itself. Supply a suitable seed or Random Float range; its selection uses the integer part of the input, so a `0` to `1` random range is usually unsuitable.

See the [selector and variation examples](/animation/animgraph/runtime-graph-builder/clips-and-blends.md#select-once-on-entry).

These entry-time choices are different from [named graph variations](/animation/animgraph/variations.md), which share graph structure but override authored node data.

## State machines and transitions

| Node | What it does / where it belongs |
|---|---|
| State Machine | Runs states containing pose graphs and blends between them according to transition conditions. Can contain nested machines. |
| Transition | Defines one conditional route inside a transition conduit, including its blend and timing settings. An unconnected condition disables it. |
| Global Transition | Defines a route to one target from other states. An explicit conduit to that target replaces the global route for that source. Managed through the state-machine editor. |
| Entry State Conditions | Chooses the entry state when the machine initializes. Does not switch an already-running machine. Editor-managed. |
| State Completed | Tests completion inside a transition conduit. Normally triggers early enough for the transition blend to finish at the state's end; At Clip End waits until the end. |
| Time Condition | Compares playback time, time in state, state progress, or sync-event progress. Use inside a transition conduit. |
| State Layer Data | Supplies a state's layer weight, root-motion weight, and bone mask when the machine is used as a layer. Editor-managed. |

**Off State** is a state-machine state type, not a separate pose node. It disables an optional action layer without putting an idle pose over the base.

See [States and Transitions](/animation/animgraph/states-and-transitions.md) for an idle/wave example, and [runtime state machines](/animation/animgraph/runtime-graph-builder/state-machines.md) for C# construction.

## Graph composition

| Node | What it does / when to use it |
|---|---|
| Pose Group | Organizes a pose tree inside a named group with exposed inputs. Use it to keep a larger graph readable. |
| Group Input | Exposes a typed connection inside a Pose Group. Must belong to a group, not the top-level graph. |
| [Internal Graph](/animation/animgraph/internal-graphs.md) | Runs a fixed child `.ag` asset and reflects compatible matching parent parameter values into it. Called `ReferencedGraph` in the builder. |
| External Graph | Declares a named slot that gameplay can connect to a chosen graph at runtime. Uses its fallback while disconnected. |
| Is External Slot Filled | Tests whether a graph is connected to an External Graph slot. Connected does not necessarily mean its branch is active. |
| External Pose | Accepts a pose supplied by native integration. It is not clip playback, and managed runtime pose assignment is not exposed. |

Unlike Internal Graph, External Graph does not inherit parent control values. See [External Graphs](/animation/animgraph/external-graphs.md) for custom item hold types, editor wiring, layered actions, connection rules, and complete gameplay examples.

## Parameters

Define parameters in the **Parameters** panel, then drag them into the graphs that use them. These nodes read the named value; they do not automatically calculate gameplay speed, grounded state, or targets.

| Node | Value it provides |
|---|---|
| Bool Parameter | A true/false control or virtual parameter. |
| Float Parameter | A numeric control or virtual parameter. |
| ID Parameter | A named-choice control or virtual parameter, such as a hold type. |
| Vector Parameter | A three-component control or virtual parameter. Its meaning depends on the consuming node. |
| Target Parameter | A target control or virtual parameter carrying a transform or bone reference. |
| Bone Mask Parameter | A virtual bone-mask value. Bone masks are not externally settable control parameters. |

Control parameters come from gameplay; virtual parameters name calculations inside the graph. Their reference nodes can display the parameter's name rather than the generic type above.

See [Parameters and Code](/animation/animgraph/parameters-and-code.md).

## Bool and float values

| Node | What it does / when to use it |
|---|---|
| Bool | Outputs a constant true or false. |
| Float | Outputs a constant number. |
| Variation Float | Outputs a constant number that can differ per graph variation. It is not a live gameplay parameter. |
| Random Float | Samples a number in its configured range on initialization, then holds it until shutdown. |
| And | Returns true only when every connected Bool input is true. |
| Or | Returns true when either Bool input is true. |
| Not | Inverts a Bool. |
| Float Comparison | Compares a float against another value. Near-equal uses an Epsilon tolerance. |
| Float Range Check | Tests whether a float is within the configured range. |
| Float Math | Applies arithmetic, rounding, sign, or fractional operations. |
| Clamp | Limits a float to the configured minimum and maximum. |
| Remap | Maps between input and output ranges, clamping to their endpoints. |
| Angle Math | Wraps or flips an angle in degrees. |
| Curve | Maps a float through an authored curve. |
| Ease | Smooths toward a changing input over a chosen time and easing curve. |
| Spring | Follows a changing input with a damped spring. Useful for smoothing movement values; it can overshoot. |
| Float Switch | Chooses configured true/false values from a Bool, optionally easing between them. |
| Float Selector | Uses the first true condition's value, or the default, and can ease between choices. Evaluates choices continuously. |

Float Selector is a **value** selector, unlike the entry-only pose Selector. See [Values and Parameters](/animation/animgraph/runtime-graph-builder/values-and-parameters.md) for calculations and smoothing examples.

## ID values

IDs are names, not numeric animation indices.

| Node | What it does / when to use it |
|---|---|
| ID | Outputs a constant ID. |
| ID Comparison | Checks whether an input ID matches a configured list, or inverts that check. |
| Variation ID Comparison | Compares the input with an ID list that can differ per graph variation. It does not simply test the current variation's name. |
| ID Switch | Chooses between two ID values according to a Bool. |
| ID Selector | Continuously chooses the ID of the first true condition, or the default ID. |
| ID To Float | Maps a named choice to a number, with a default for unmatched IDs. |

ID Selector chooses an **ID value** continuously. ID Based Selector chooses a **pose** on entry.

## Vectors and targets

| Node | What it does / when to use it |
|---|---|
| Vector | Outputs a constant three-component vector. |
| Vector Create | Builds a vector from Float components, optionally using another vector for unspecified components. |
| Vector Info | Extracts a component, length, or horizontal/vertical angle from a vector. |
| Vector Negate | Reverses the sign of every vector component. |
| Transform Target | Produces a set target with a fixed position and rotation. |
| Bone Target | Produces a target referring to a character bone. |
| Is Target Set | Tests whether a target is set. |
| Target Info | Extracts a distance, angle, or orientation difference relative to the character. |
| Target Point | Extracts a target's position as a Vector. |
| Target Offset | Adds position and rotation offsets to a target, optionally in bone space. |

A Vector point and a Target transform are not interchangeable. Match world-space or character-space interpretation to the consuming node. See [target calculations](/animation/animgraph/runtime-graph-builder/values-and-parameters.md#targets).

## Bone masks

| Node | What it does / when to use it |
|---|---|
| Bone Mask | Selects an authored skeleton mask, or uses Default Weight uniformly when none is selected. |
| Fixed Weight Bone Mask | Gives every bone the same configured weight. |
| Bone Mask Blend | Blends between source and target masks using a Float weight. |
| Bone Mask Selector | Selects a mask by ID, with a default mask when no option matches. |
| Bone Mask Switch | Selects the true or false mask according to a Bool. |

For Bone Mask Selector and Bone Mask Switch, enable **Switch Dynamically** if the mask should respond to input changes while active. Otherwise selection happens on entry. Blend Time controls changes when dynamic switching is enabled.

See [Layers and Masks](/animation/animgraph/layers-and-masks.md).

## IK and aiming

| Node | What it does / when to use it |
|---|---|
| Aim At | Aims a weapon bone at a world-space point using `weaponTip` and `weaponEnd` helper bones. Follow with hand IK where appropriate. |
| Look At | Distributes pitch/yaw across a weighted bone chain. Configure its reference bone and order the chain end-to-root. |
| Two Bone IK | Rotates an effector's two parents to reach a Target, such as positioning a hand. |
| Foot IK | Applies two-bone IK to one or both feet using supplied targets. Does not perform ground traces for you. |

See [Procedural Animation](/animation/animgraph/runtime-graph-builder/procedural-animation.md) for target spaces, chain setup, and examples.

## Root motion

| Node | What it does / when to use it |
|---|---|
| Orientation Warp | Redirects a clip's authored root motion using an orientation-warp event and a direction or angle input. |
| Target Warp | Adjusts a clip's root motion to reach a world-space target transform using authored target-warp events. |
| Root Motion Override | Replaces movement velocity, facing direction, or both with character-space gameplay values. |

Warps require clip-specific inputs and suitable authored motion/events. Producing root motion is separate from moving a GameObject with collision handling. See [warping and root-motion overrides](/animation/animgraph/runtime-graph-builder/procedural-animation.md#warp-authored-root-motion).

## Events and synchronization

These nodes query sampled animation data inside the graph. They do not register C# callbacks, and a true condition is not automatically a one-shot notification.

| Node | What it provides |
|---|---|
| Footstep Condition | Whether a sampled foot event matches a selected foot phase. |
| Footstep Event ID | The sync ID carried by the selected sampled foot event. |
| Footstep Percentage Through | Progress through a matching foot event, or `-1` if none matches. |
| ID Event Condition | Whether sampled graph/ID animation events contain the requested IDs. |
| ID Event Value | The selected sampled ID animation event's ID, or a configured default. |
| ID Event Percentage Through | Progress through a matching ID animation event, or `-1` if absent. |
| Graph Event Condition | Whether sampled graph events match the requested IDs and event types. |
| Float Curve Event | A value sampled from a float-curve animation event, or a fallback. Useful for authored IK or layer weights. |
| Transition Event Condition | Whether sampled transition-permission events satisfy a rule, such as allowing interruption. |
| Sync Event Index Condition | Tests the source state's current sync-event index. Use inside a transition conduit. |
| Current Sync Event ID | The source state's current sync-event ID. Use inside a transition conduit. |
| Current Sync Event Index | The source state's sync-event index, percentage, or both, according to its settings. Use inside a transition conduit. |

For event types, clip authoring, and C# callbacks, see [Animation Events](/animation/animgraph/events.md). Scope event queries deliberately: during blending, more than one clip or layer can contribute events. See [Events and Timing](/animation/animgraph/runtime-graph-builder/events-and-timing.md).

## Cached values

Use these when a branch should hold a value rather than continuously follow its source.

| Node | Value type |
|---|---|
| Cached Bool | True/false |
| Cached Float | Number |
| Cached ID | Named choice |
| Cached Vector | Three-component vector |
| Cached Target | Animation target |

**On Entry** samples once when initialized. **On Exit** follows the input while active, then retains the final value when the branch deactivates. The node's initialization lifetime matters; a virtual parameter kept alive for the whole graph will not restart just because a state changes.

See [virtual parameters and cached values](/animation/animgraph/runtime-graph-builder/values-and-parameters.md#virtual-parameters-and-cached-values).

## Outputs and editor helpers

These are graph structure or editor tools, rather than additional animation effects.

| Node | What it does / where it belongs |
|---|---|
| Result (pose) | Receives a pose graph's output. This is the graph output described in the tutorials; it also appears inside pose subgraphs. Editor-managed. |
| Result (virtual parameter) | Receives the Bool, Float, ID, Vector, Target, or BoneMask produced by a virtual parameter's value graph. Editor-managed. |
| Conditions | Holds the condition inputs inside a Selector's condition graph. Editor-managed. |
| Reroute | Changes how a connection is drawn without changing its value or pose. |
| Comment | Labels and groups related logic visually. Has no runtime effect. |
