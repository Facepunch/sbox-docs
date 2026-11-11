---
title: "Animation Events"
created: 2026-11-11
updated: 2026-11-11
---

# Animation Events

Animation events mark moments or sections of a clip: a magazine being inserted, a foot touching the ground, or a window where an action can be interrupted.

**Choose the event by who needs to read it.** A named ID for the graph is not the same as a typed event delivered to game code.

| I want to... | Use | Read it with |
|---|---|---|
| Notify game code at an authored point, with custom properties | A **typed event** on an Animation Events track | `Component.IAnimationEvent` |
| Mark a moment or window for graph logic | An **ID event** on an ID Events track | ID Event Condition, ID Event Value, ID Event Percentage Through |
| Describe foot phases for locomotion | A **foot event** on a Foot Events track | Footstep Condition, Footstep Event ID, Footstep Percentage Through |
| Use an existing engine animation-event handler | A **native event** on an Animation Events track | The handler for that native event name and payload |
| Report state entry, exit, or another graph-level condition | A **graph event**, configured on the state or node | Graph Event Condition, ID Event Condition, or `OnAnimTagEvent` |

Typed and native events share the Animation Events track. **Add Event Here** creates a typed event; **Add Native Event Here** creates a native one. ID and Foot Events are separate track types.

## Author events on a clip

Open an `.anim` asset from the Asset Browser. Its **Timeline** contains the event tracks, and **Event Properties** edits the selected event.

1. Use **Add a track** to choose **Animation Events**, **ID Events**, or **Foot Events**.
2. Select the track and move the playhead to the desired frame.
3. Use the **+** button to add an event at the playhead, or right-click the lane and choose **Add Event Here**.
4. Select the event and set its type, ID, or phase in Event Properties.
5. Set its **Frames** range and save the clip.

Drag an event to move it. Drag a duration event's ends to resize it. Frame snapping is on by default; hold **Ctrl** while dragging to bypass it.

A point event has the same start and end frame and appears as a marker. A duration event spans a range and appears as a block.

The clip must compile with at least two frames to edit its timeline. A track marked **Disabled** keeps its authored events but excludes them from compilation.

## Typed events: notify game code

A typed event is an instance of a game-defined class derived from `Sandbox.Animation.AnimationEvent`. Its `[Property]` members appear in Event Properties.

For example, define a cue for the moment a magazine is inserted:

```csharp
using Sandbox;
using Sandbox.Animation;

[Title( "Magazine Inserted" )]
public sealed class MagazineInsertedEvent : AnimationEvent
{
    [Property] public string SoundName { get; set; } = "magazine_insert";
}

public sealed class WeaponAnimationEvents : Component, Component.IAnimationEvent
{
    public void OnAnimationEvent( AnimationEvent data )
    {
        if ( data is not MagazineInsertedEvent magazine )
            return;

        Log.Info( $"{magazine.Renderer.GameObject.Name}: {magazine.SoundName}" );
    }
}
```

Once the game code compiles, add an event to an **Animation Events** track. Select **Magazine Inserted** in its **Type** picker, set Sound Name, and position the event at the insertion frame.

Add `WeaponAnimationEvents` to the GameObject containing the `SkinnedModelRenderer`, or to one of its parents. During playback, an active receiver gets `OnAnimationEvent`; `data.Renderer` identifies the renderer that fired it. The example logs the cue: the string property does not play a sound automatically.

You do not need to register a callback or assign a renderer property on the receiver. A receiver on a sibling or child GameObject does not receive events from that renderer.

Treat the payload as **read-only**. Other receivers get the same instance, so changing its properties inside a handler changes what they see too.

### Read typed events without playback

`Clip.GetEvents<T>()` reads the authored typed events and their timing:

```csharp
var clip = Clip.Load( "animations/reload.anim" );
foreach ( var entry in clip.GetEvents<MagazineInsertedEvent>() )
{
    Log.Info( $"{entry.StartTime:0.00}s: {entry.Data.SoundName}" );
}
```

This inspects the clip; it does not play it or dispatch callbacks. StartTime, Duration, and EndTime are in seconds at the clip's original playback speed. It excludes ID, foot, and native events. The payload's Renderer is unset when reading authored events this way.

## ID events: label moments and windows for the graph

An ID event carries a string such as `can_interrupt`. It has no C# event class or custom payload.

For an interruption window:

1. Add an **ID Events** track to the action clip.
2. Add an event with ID `can_interrupt` and extend it over the interruptible frames.
3. In the action's transition graph, add **ID Event Condition**.
4. Add `can_interrupt` to **Event Ids** and set **Search Rule** to **Only Search Anim Events**.
5. Combine its result with the gameplay request using **And**, then connect that to **Transition.Condition**.

The condition is true while the event is sampled. It does not create a transition or interrupt the action by itself. In a transition, enable **Limit Search To Source State** when the marker must come from the state being left.

**ID Event Value** returns a sampled ID. **ID Event Percentage Through** returns progress through a matching event, or `-1` when none matches. A range is useful for a window; a point is useful for a moment.

An ID event named `can_interrupt` does **not** call `OnAnimationEvent`, `OnGenericEvent`, or a C# method of that name. It is also not an ID control parameter.

## Foot events: describe the locomotion phase

Foot events carry a **Phase**: Left Foot Down, Right Foot Passing, Right Foot Down, Left Foot Passing, or None.

Place these on a **Foot Events** track to describe where the feet are in the cycle. Graph nodes can then wait for a planted foot, read progress through a phase, or synchronize locomotion clips.

Despite the node names, a foot-phase event does **not** automatically play a footstep sound or invoke `SkinnedModelRenderer.OnFootstepEvent`. That callback handles the separate native `AE_FOOTSTEP` event. Use a typed event if you want your own game-code cue at foot contact.

## Native events: existing engine names and payloads

**Add Native Event Here** creates an event whose **Name** is passed to the engine's animation-event dispatch. These are for existing native event conventions, including imported sound and footstep events.

| Native name | Payload fields | Runtime handling |
|---|---|---|
| `AE_FOOTSTEP` | `Foot`, `attachment`, `volume` | Invokes `OnFootstepEvent` |
| `AE_CL_PLAYSOUND`, `AE_CL_PLAYSOUND_ATTACHMENT`, `AE_CL_PLAYSOUND_POSITION`, `AE_SV_PLAYSOUND` | `name`, optional `attachment` | Invokes `OnSoundEvent` and plays the sound through the renderer |
| `AE_GENERIC_EVENT` | `TypeName`, `Int`, `Float`, `StringData`, `Vector` | Invokes `OnGenericEvent` |
| `AE_CL_BODYGROUP_SET_VALUE` | `bodygroup`, `value` | Sets the named bodygroup's value |

**A native event needs the payload its handler expects, not just its name.** The current native-event properties expose the name and frame range; existing payload data is preserved, but this is not a custom C# payload editor.

Naming a native event `native_event` or `reload` does not register a new handler, invoke a matching C# method, or make it arrive through `OnGenericEvent`. That callback specifically handles `AE_GENERIC_EVENT`, whose payload supplies TypeName, Int, Float, StringData, and Vector.

For new game-defined behavior, use a **typed event** instead. Typed events use the reserved native name `AE_MANAGED_EVENT` internally; create them through the Type picker rather than entering that name as a native event.

## Graph events: state and node activity

Graph events belong to the graph, not to frames authored on an `.anim` timeline.

A state's **Events** can report entry, fully-in-state, and exit. **Timed Events** report authored state-time conditions. Animation Clip nodes also have a **Graph Events** list.

Use **Graph Event Condition** to match a graph event's ID and type. **ID Event Condition** can search graph events, clip ID events, or both, according to Search Rule.

Game code can observe graph-event names through `SkinnedModelRenderer.OnAnimTagEvent`. For AG2, it reports **Start** when a named graph event becomes active and **End** when it stops being active. It is not the typed-event interface, and it does not report clip ID events.

## Sync tracks are not another callback type

Right-click a track header and enable **Sync Track** to use that track's events to divide the clip into phases. A clip has only one sync track.

Use ID or foot tracks with corresponding phases across locomotion clips, so synchronized blends can align those phases rather than only matching normalized playback time. Setting Sync Track does not turn an ID or foot event into a game-code callback.

Keep sync markers distinct: markers at the same or nearly the same normalized time can prevent the clip from compiling. The timeline warns about this.

## Timing and blending

Use a **point event** for a one-off cue. A duration event remains sampled over its range; typed and native callbacks can repeat on successive animation updates while that range is sampled. A duration does not automatically give your handler separate start and end notifications.

Loops, reverse playback, and re-entering a state can encounter an event again. **Animation Pose** only samples a pose and does not produce playback events.

Typed and native callbacks are filtered by graph contribution: the event must come from an active, non-ignored branch and its accumulated sampled weight must be **greater than `0.5`**. A clip contributing weakly to a blend may therefore produce no callback. A layer's **Ignore Events** setting can also suppress events.

Keep authoritative gameplay and networking in game code; animation playback is not a guarantee that a callback will happen exactly once.

For graph-query settings and C# builder examples, see [Events and Timing](/animation/animgraph/runtime-graph-builder/events-and-timing.md). For direction-specific sampling, see [Reversing Clips](/animation/animgraph/reversing-clips.md).
