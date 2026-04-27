---
title: Input, Navigation & AI
icon: "🧭"
created: 2026-04-25
updated: 2026-04-25
---

# Input, Navigation & AI

Input not registering, NavMeshAgent not pathing, AI flickering between states, VR tracking issues.

## Input

### "My Input.Down('fire') is never true"

Action name doesn't match what's defined in Project Settings, or no key is bound to it.

- **Check:** action name spelling (case-insensitive but must match exactly otherwise).
- **Check:** the action has at least one key/button binding in Project Settings.

**Deep dive:** [Input Index](../systems/input/index.md), [Detect Player Input](../how-to/detect-input.md).

### "Input.Pressed fires multiple times in a frame"

`Input.Pressed` returns `true` for the **entire frame** the press occurred. Inside a per-frame loop or multi-callback path, you'll see it multiple times.

- **Fix:** call `Input.Pressed` once per frame at the top of your update, store the result.

### "Glyph shows as UNBOUND"

Action is unbound, or you misspelled the action name.

- **Fix:** confirm the action exists in Project Settings and at least one binding is configured.

**Deep dive:** [Glyphs](../systems/input/glyphs.md).

### "I'm hardcoding for a key, why isn't it consistent?"

You're using raw key names (e.g., "E", "Space") instead of actions. Raw keys break controllers and rebinding.

- **Fix:** define an action in Project Settings and use `Input.Down("my_action")`. Always prefer actions.

**Deep dive:** [Raw Input](../systems/input/raw-input.md).

### "Raw Input: key never registers"

Key name string doesn't match the engine's expected case.

- **Fix:** use exact strings from the input table (e.g., `"Space"`, not `"SPACE"`).

### "Input.UsingController seems wrong"

`Input.UsingController` is dynamic — it flips when the user uses mouse vs controller.

- **Fix:** don't cache the value at start; query each frame.

**Deep dive:** [Controller Input](../systems/input/controller-input.md).

## Navigation / NavMesh

### "My agent isn't moving to the target"

No NavMesh baked in the scene.

- **Fix:** generate the NavMesh in the scene. Visualize it in the editor to confirm coverage.
- **Check:** the target position is on the navmesh — `MoveTo` silently returns if the nearest poly query fails.

**Deep dive:** [Navigation Index](../systems/navigation/index.md), [NavMesh Agent](../systems/navigation/navmesh-agent.md).

### "Agent is floating or sinking into the floor"

NavMesh has lower height resolution than the actual ground.

- **Fix:** adjust the agent's `Height` and `BaseOffset` properties to compensate.

### "Agent jitters or fights my custom controller"

Agent is updating the GameObject's position, while you're also moving it from another component.

- **Fix:** set `UpdatePosition = false` on the agent if you're driving the transform yourself (e.g., with `CharacterController`).

### "Agent ignoring an Area's cost"

`Is Blocker` is true on the area — that overrides cost and treats it as an obstacle.

- **Fix:** set `Is Blocker = false`. Raise `CostMultiplier` if the agent still routes through it.

**Deep dive:** [Costs & Filters](../systems/navigation/navmesh-areas/costs-filters.md).

### "Obstacle isn't blocking the navmesh"

The `SceneVolume` doesn't intersect the navmesh.

- **Fix:** lower the volume so it cuts into the floor. Floating volumes don't carve.

**Deep dive:** [Obstacles](../systems/navigation/navmesh-areas/obstacles.md).

### "AI gets stuck on corners or walls"

`Radius` and `Height` on the `NavMeshAgent` are smaller than the visual model — it tries to squeeze through gaps that don't fit.

- **Fix:** match Radius/Height to the model footprint.

## State machine AI

### "AI keeps twitching between two states"

Your transition conditions overlap with no deadband. A unit standing exactly at distance 500 oscillates if you switch to Chase at `<500` and Idle at `>500`.

- **Fix:** add hysteresis: enter Chase at `<500`, leave Chase at `>600`.

**Deep dive:** [State Machine AI](../how-to/state-machine-ai.md).

## VR

### "VR Hands stuck at 0,0,0"

No `VRAnchor` in the scene. Without it, local VR coordinates can't be translated to world space.

- **Fix:** add a GameObject with a `VRAnchor` component in your active scene.

**Deep dive:** [VR Anchor](../systems/vr/vranchor.md), [VR Input](../systems/input/vr-input.md).

### "Camera renders but doesn't track my head"

`VR Tracked Object` component is missing from the camera GameObject.

- **Fix:** add `VRTrackedObject` and set its `TrackedItem` to `Head`.

**Deep dive:** [VR Index](../systems/vr/index.md).

### "Hands are floating far from the body"

`VRTrackedObject.UseRelativeTransform` is unchecked — hands track relative to world origin instead of the anchor.

- **Fix:** tick `Use Relative Transform` so hands move with the anchor.

**Deep dive:** [VR Tracked Object](../systems/vr/vrtrackedobject.md).

### "VR fingers aren't moving (only the hand position is)"

`VRHand` only animates fingers; it doesn't position the hand.

- **Fix:** add a `VRTrackedObject` to set hand position. Use `SkinnedModelRenderer` (not `ModelRenderer`) so the model can pose.

**Deep dive:** [VR Hand](../systems/vr/vrhand.md).

### "Showing a default box instead of my controller model"

SteamVR didn't provide a model for the connected hardware.

- **Fix:** ensure the VR runtime is properly configured. Provide a fallback `Model` if you want a controlled appearance.

**Deep dive:** [VR Model Renderer](../systems/vr/vrmodelrenderer.md).

### "VRAnchor doubled — hands and camera fight"

You have multiple active `VRAnchor` components in the scene.

- **Fix:** only one `VRAnchor` should be active per scene.

### "VR isn't launching at all"

VR runtime not configured before the editor/game started.

- **Fix:** plug in headset, start SteamVR (or your VR runtime), **then** launch s&box.
- **Fix:** verify VR.IsActive at runtime; hardware/runtime issues bubble up here.

### "Input.VR throws exceptions"

You're accessing VR data while the game isn't running in VR.

- **Fix:** check `Game.IsRunningInVR` before reading `Input.VR.*`.

### "VR Controller returns default/zero values"

Controller is asleep, off, or out of tracking range.

- **Fix:** write defensive code — check controller validity before using values.

**Deep dive:** [Platform VR Input](../systems/platform/vr-input.md).

## ActionGraph (visual scripting)

### "My ActionGraph isn't doing anything"

Nodes aren't connected to the white signal wire from the Root.

- **Fix:** drag from the Root's white output through your action chain.
- **Check:** the GameObject containing the ActionGraph component is enabled.

**Deep dive:** [ActionGraph](../systems/actiongraph/index.md).

### "Set Variable doesn't change my variable"

`Set Variable` is an action node — needs a white execution signal.

- **Fix:** connect the white signal wire through the Set node.

**Deep dive:** [ActionGraph Variables](../systems/actiongraph/variables.md).

### "Pure node doesn't run when expected"

`[Pure]` nodes only evaluate when their outputs are read by something downstream.

- **Fix:** if the method has side effects (changes game state), don't mark it `[Pure]`.

**Deep dive:** [C# Method Nodes](../systems/actiongraph/custom-nodes/c-method-nodes.md).

## Animations and IK

### "Direct Playback won't play my animation"

`DirectPlayback.Play()` takes a **Sequence** name from the model, not an AnimGraph state.

- **Fix:** open the model in Model Editor → list of sequences. Use one of those names.

**Deep dive:** [Animate a Model](../how-to/animate-model.md).

### "AnimGraph parameters aren't doing anything"

`UseAnimGraph` is unchecked on the `SkinnedModelRenderer`, or the model has no AnimGraph assigned.

- **Check:** `UseAnimGraph` is ticked.
- **Check:** the `.vmdl` actually has an AnimGraph assigned in its asset settings.

### "Character is T-posing"

Same family — `UseAnimGraph` off, or no AnimGraph on the model.

**Deep dive:** [Skinned Model Renderer](../scene/components/reference/skinnedmodelrenderer.md).

### "SetIk does nothing"

The model's AnimGraph must have an IK node (Two-Bone IK or similar) wired to the IK parameters.

- **Fix:** open the AnimGraph and add an IK node that consumes the parameters `SetIk` writes.

**Deep dive:** [Use IK](../how-to/use-ik.md).

### "IK limb snaps wildly"

You passed a local-space transform. `SetIk` expects **world-space**.

- **Fix:** pass world transforms; the engine handles the local conversion internally.

### "Weapon attachment is offset from the bone"

After parenting, you didn't reset `LocalPosition`/`LocalRotation`.

- **Fix:** after `SetParent`, set `LocalPosition = Vector3.Zero` and `LocalRotation = Rotation.Identity` to snap to the bone.

**Deep dive:** [Attach an Object to a Bone](../how-to/attach-weapon.md).

### "GetBoneObject returns null"

Bone name typo — they're case-sensitive and must exactly match the model's bone names.

- **Fix:** open the model in Model Editor and copy the exact bone name.

## Related Pages

- [Scene & Components](./scene-and-components.md)
- [Networking](./networking.md)
- [Audio & Effects](./audio-and-effects.md)
