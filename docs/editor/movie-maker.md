---
title: "Movie Maker"
icon: "🎬"
sources:
  - game/editor/MovieMaker/Code/
updated: 2026-04-25
created: 2026-04-27
---

# Movie Maker

Movie Maker is s&box's built-in cinematic and sequencing tool.

1. **Animating GameObject Properties:** You can drag a GameObject from the Scene Tree into the Movie Maker timeline to create a track for it. You can then add tracks for its position, rotation, or any exposed `[Property]` on its components.
2. **Camera Cuts:** You can create a "Camera Track" to sequence cuts between multiple different cameras in the scene, effectively editing a movie.
3. **Triggering Code:** You can add event tracks that fire C# methods at specific points in the timeline.

## How to use Movie Maker

1. Open the s&box Editor.
2. Go to `View -> Movie Maker` (or look for the Movie Maker icon in the toolbar).
3. Create a new Movie sequence.
4. Drag objects from your scene into the timeline on the left side to begin tracking them.
5. Move the playhead (the vertical line in the timeline) to a specific time.
6. Change the property of the object in the inspector.
7. Click the "Keyframe" button next to the property in Movie Maker to lock in that value at that specific time.

## Troubleshooting

:::warning "My animation doesn't play in the game!"
Ensure you have added a `MoviePlayer` component to a GameObject in your scene and assigned your `.movie` asset to its **Movie** field (the `Resource` property). To start automatically when the scene loads, tick **IsPlaying** on the component — there is no separate "Play on Start" toggle.
:::

:::warning "Missing Object Bindings"
If you rename or delete a GameObject that is referenced in a `.movie` asset, the tracks targeting that object will fail to resolve. You will need to re-bind the track to the new object in the Movie Maker interface.
:::
