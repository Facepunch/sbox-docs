---
title: "Getting Started"
icon: "🌱"
created: 2025-06-30
updated: 2025-08-27
---

# Getting Started

In this article you'll find out:

* [How to open the Movie Maker editor](#h-movie-editor)
* [What is a Movie Player](#h-movie-player)
* [What is a Movie Resource](#h-movie-resources)
* [What is a Track](#h-tracks)

# Movie Editor

## Opening the Movie Editor

If Movie Maker isn't already visible, you can toggle it in the View menu.

[sbox-dev_xERW1ujnPd.mp4 2098x1076](./images/fe0e9321-d2d7-45d7-957b-552c67212c72.png)

It should fit nicely as a tab in the lower panel if you dock it there.


# Movie Player

To edit and preview movies, you'll need a Movie Player component somewhere in your scene. Movie Players decide which objects in the scene should be animated by a particular movie, and control the current playback position.

##  ![](./images/9e4b8702-9b7f-4d86-8bb5-bc2c41ce4d16.png "right-50 =528x369")Creating a Movie Player

If you don't have any Movie Players in the current scene, you should see a big button to create one.

Otherwise, you can switch between players or create new ones by using the drop-down in Movie Maker's menu bar. This will only list players in the currently active scene.

You can also add one to a GameObject in the Inspector like any other Component.

# Movie Resources

Movies can either be embedded inside a Movie Player, or saved as a *.movie* asset.

## Saving & Loading ![](./images/7e86e92d-5f63-4e02-ad0b-08525842d54c.png "left-50 =486x351")

New movies will be embedded in the current Movie Player by default. You can save them as reusable .movie files, or embed a copy of the currently edited .movie, with the File menu.

## Sequences

You can also reference segments of movies from each other using sequence blocks. This can be done by selecting *Import Movie* in either the file menu or when right-clicking in the timeline.


# Tracks

Movies describe how properties in a scene should change over time, and this information is stored in tracks.

There's a few types of track you'll need to know about.

### Reference Track

This track references a GameObject or Component in the scene that should be controlled. It's up to the Movie Player to decide which particular object to bind each track to, so you can re-use the same .movie to control different actors.

### Property Track

This represents a property somewhere in the scene, and describes how it should animate. This could be the position of the camera, the colour of a light, or text in a speech bubble.

Property tracks are always nested inside other tracks, either reference tracks or other properties. This is how the track knows what to control in the scene: it checks what the parent track is bound to, and looks up a property inside it with the track's name.

### Sequence Track

This track references blocks of time from another movie, helping you organize and edit bigger movie projects with multiple shots.

## Creating Tracks

Reference and property tracks can be created by dragging from the hierarchy or inspector into the track list.

[sbox-dev_1Mc8djxfBo.mp4 1492x948](./images/cf51600c-93f0-4a11-b46d-d9df71dd0228.png)

You can also create sub-tracks by right-clicking an existing track, and selecting the tracks you want from the context menu.


 ![](./images/271bebe9-c6b7-49a5-b5e3-e34a20dd0196.png " =613x212")

Sequence tracks are created automatically when you import a movie: either by right-clicking in the timeline, or through the file menu.


 ![](./images/e8057da3-2d17-411b-a933-9b43839c580a.png " =784x475")

# Next Steps

* [Editor Map](/systems/movie-maker/editor-map.md) - find your way around the movie editor
* [Keyframe Editing](/systems/movie-maker/keyframe-editing.md) - a great place to start making simple animations
* [Motion Editing](/systems/movie-maker/motion-editing.md) - make more detailed animations and edit recordings
* [Recording](/systems/movie-maker/recording.md) - manually puppet characters or record gameplay
* [Sequences](/systems/movie-maker/sequences.md) - cleanly structure big projects with nested movies
* [Playback API](/systems/movie-maker/playback-api.md) - define movies and play them back using C#.
* [Recording API](/systems/movie-maker/recording-api.md) - record and save the gameplay using C#